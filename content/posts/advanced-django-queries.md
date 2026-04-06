---
title: "Advanced Django queries"
draft: false
date: "2021-07-27T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["django", "python", "sql"]
description: "Django Framework comes with a powerful ORM and query capabilities built-in. If you're only familiar with the basics of Djangos Query API, this article will introduce some more advanced queries and methods you can use."
---

Django Framework comes with a powerful ORM and query capabilities built-in. If you're only familiar with the basics of Djangos [Query API](https://docs.djangoproject.com/en/3.2/ref/models/querysets/#queryset-api), this article will introduce some more advanced queries and methods you can use.

In the examples, I'll be using the following data models.

```python
class Author(models.Model):
    nickname = models.CharField(max_length=20, null=True, blank=True)
    firstname = models.CharField(max_length=20)
    lastname = models.CharField(max_length=40)
    birth_date = models.DateField()


class Book(models.Model):
    author = models.ForeignKey(Author, related_name="books", on_delete=models.CASCADE)
    title = models.CharField(unique=True, max_length=100)
    category = models.CharField(max_length=50)
    published = models.DateField()
    price = models.DecimalField(decimal_places=2, max_digits=6)
    rating = models.IntegerField()
```

### Filtering

Basic filtering in Django can be done using something called _field lookups_ with the `filter` method. Field lookups consist of the field and a suffix defining the lookup type. If no suffix is defined, the default behavior is equivalent to using the [exact](https://docs.djangoproject.com/en/3.2/ref/models/querysets/#exact) suffix.

```python
books = Book.objects.filter(title="Crime and Punishment")
books = Book.objects.filter(title__startswith="Crime")
```

Querying by using field lookups is already very powerful. We can filter with different suffixes like [gte](https://docs.djangoproject.com/en/3.2/ref/models/querysets/#gte), [lte](https://docs.djangoproject.com/en/3.2/ref/models/querysets/#lte), [contains](https://docs.djangoproject.com/en/3.2/ref/models/querysets/#contains), and a [myriad of other lookups](https://docs.djangoproject.com/en/3.2/ref/models/querysets/#field-lookups) depending on the field type. 

```python
# Books published in 2021
books = Book.objects.filter(published__year=2021)
# Books published in the year 2000 or after
books = Book.objects.filter(published__year__gte=2000)
# Books published before the year 2000
books = Book.objects.filter(published__year__lt=2000)
```

With time fields we can even filter using a `range` lookup to query objects within a specific time range.

```python
start_date = datetime.date(2021, 1, 1)
end_date = datetime.date(2021, 1, 3)
books = Book.objects.filter(published__range=(start_date, end_date))
```

Using the [built-in expressions](https://docs.djangoproject.com/en/3.2/ref/models/expressions/#built-in-expressions) and [Q objects](https://docs.djangoproject.com/en/3.2/topics/db/queries/#complex-lookups-with-q-objects) we can do some more advanced queries.

#### Q Objects

Keyword arguments in a **filter** query are "AND"ed together. If we want to execute **OR** queries we can use the [Q object](https://docs.djangoproject.com/en/3.2/topics/db/queries/#complex-lookups-with-q-objects). **Q objects** encapsulate keyword arguments for filtering just like **filter**, but we can combine **Q objects** using **&** or **|**.

```python
from django.db.models import Q
# Get all books published in 2018 or 2020
books = Book.objects.filter(Q(published__year=2018) | Q(published__year=2020))
# Get all books published in
```

This will query all books published in 2018 or 2020.

You can combine any number of **Q objects** into more complex queries.

#### F expressions

[F expressions](https://docs.djangoproject.com/en/3.2/ref/models/expressions/#django.db.models.F) represent a value of a model field. It makes it possible to use field values in queries without actually pulling the value from the database. This is possible because Django creates a SQL query that handles everything for us.

```python
from django.db.models import F
# Query books published by authors under 30 years old (this is not exactly true because years vary in length)
books = Book.objects.filter(published__lte=F("author__birth_date") + datetime.timedelta(days=365*30))
```

We can also use **F expressions** when updating data to avoid performing multiple queries.

```python
from django.db.models import F
book = Book.objects.get(title="Crime and Punishment")
book.update(rating=F("rating") + 1)
```

This will result in an SQL query that will add one to the rating of the book without first querying the current value.

### Annotation

Most often we just want to query the values defined in a model with some filtering criteria. But sometimes you'll be calculating or combining values that you need from the result of the query. This can, of course, be done in Python but for performance reasons, it might be worth it to let the database handle the calculations. This is where Djangos [annotations](https://docs.djangoproject.com/en/3.2/ref/models/expressions/#using-f-with-annotations) come in handy.

```python
from django.db.models import F, Value as V
from django.db.models.functions import Concat
author = Author.objects.annotate(full_name=Concat(F("firstname"), V(" "), F("lastname")))
```

Here, we are adding a new dynamic field `full_name` to our query results. It will contain a concatenation of the authors **firstname** and **lastname** done using the [Concat](https://docs.djangoproject.com/en/3.2/ref/models/database-functions/#concat) function, **F expressions**, and **Value**.

**Concat** is just one example of a [database function](https://docs.djangoproject.com/en/3.2/ref/models/database-functions/#) available in Django. **Database functions** are a great way to add dynamic fields into our queries. Django supports a number of database functions like [comparison and conversion functions](https://docs.djangoproject.com/en/3.2/ref/models/database-functions/#comparison-and-conversion-functions) (Coalesce, Greatest, ...), [math functions](https://docs.djangoproject.com/en/3.2/ref/models/database-functions/#math-functions) (Power, Sqrt, Ceil, ...), [text functions](https://docs.djangoproject.com/en/3.2/ref/models/database-functions/#text-functions) (Concat, Trim, Upper, ...), and [window functions](https://docs.djangoproject.com/en/3.2/ref/models/database-functions/#window-functions) (FirstValue, NthValue, ...)


Here's an example on how we can use **Coalesce** to get either the **nickname** or the **firstname** of a author.
```python
from django.db.models import F
from django.db.models.functions import Coalesce
books = Author.objects.annotate(known_as=Coalesce(F("nickname"), F("firstname")))
```

Now our results have a new field `known_as`, that holds the value we wanted for each **Author** in the result set.

Using **F expressions** we can also do basic arithmetic to calculate the value for a dynamic field.

```python
# Add  a new field with the authors age at the time of publishing the book
books = Book.objects.annotate(author_age=F("published") - F("author__birth_date"))
# Add a new field with the rating multiplied by 100
books = Book.objects.annotate(rating_multiplied=F("rating") * 100)
```

### Aggregation

While annotate can be used to add new values to the returned data, [aggregation](https://docs.djangoproject.com/en/3.2/topics/db/aggregation/) can be used to derive values by summarizing or **aggregating** a result set. The difference between _aggregation_ and _annotation_ is that _annotating_ adds a new field to every row of a result set and _aggregating_ reduces the results into a single row with the aggregated values. 

Common uses for aggregation are counting, averaging, or finding maximum or minimum.

```python
from django.db.models import Avg, Max, Min
result = Book.objects.aggregate(Avg("price"))
# {'price__avg': Decimal('13.50')}
result = Book.objects.aggerate(Max("price"))
# {'price__max: Decimal('13.50')}
result = Book.objects.aggerate(Min("published"))
# {'published__min': datetime.date(1866, 7, 25)}
```

Aggregation can also be done without the **aggregate** method by using the **database functions** we talked about earlier. This way we can add a dynamic field into our results with **annotate** and calculate the value for it with **aggregation**. 😎

```python
from django.db.models import Count
authors = Author.objects.annotate(num_books=Count("books"))
```

This will add a dynamic field **num_books** to every row of the result set with the number of books the author has.

Using **values**, **annotate** and a **aggregation** function allows us to do a **GROUP BY** query with a dynamic field.

```python
# Calculate average prices for books in all categories.
Book.objects.values("category").annotate(Avg("price"))
# {'category': 'Historical fiction', 'price__avg': Decimal('13.9900000000000')}, {'category': 'Romance', 'price__avg': Decimal('16.4950000000000')}
```

### Case...When

The last example I want to show is a bit more complex but showcases some of Djangos query features together. It includes using [conditional expressions](https://docs.djangoproject.com/en/3.2/ref/models/conditional-expressions/) when calculating the value of a dynamic field.

```python
from django.db.models import F, Q, Value, When, Case
from decimal import Decimal
books = Book.objects.annotate(discounted_price=Case(
    When(category="Romance", then=F("price") * Decimal(0.95)),
    When(category="Historical fiction", then=F("price") * Decimal(0.8)),
    default=None
))
```

Here we're calculating a discounted prices on all books based on the category using [Case](https://docs.djangoproject.com/en/3.2/ref/models/conditional-expressions/#case) and [When](https://docs.djangoproject.com/en/3.2/ref/models/conditional-expressions/#when)

### Further reading
[Django docs on queries](https://docs.djangoproject.com/en/3.2/topics/db/queries/)