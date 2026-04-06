---
title: "Dynamic aggregation fields in Django"
draft: false
date: "2023-09-05T09:33:01+00:00"
author: juhana.jauhiainen
tags: ["django", "python", "webdev"]
description: "Aggregation is a powerful tool when working with data in Django. With it you can have your database summarize or convert data into the format you need."
---

In this article I will walk you through on how to create dynamic aggregation in Django. By dynamic aggregation I refer to the ability to define the resulting aggregation fields programmatically.

Aggregation is a powerful tool when working with data in Django. With it you can have your database summarize or convert data into the format you need. 

Here's an example of a basic `Sum` aggregation.

```python
from django.db import models
from django.db.models import Sum

class Order(models.Model):
    total_price = models.DecimalField(max_digits=10, decimal_places=2)

# Calculate the total sum of all order prices
total_sum = Order.objects.aggregate(total=Sum('total_price'))['total']

print(f'Total sum of all order prices: {total_sum}')

```

We have a `Order` model with total price. If we want the total sum in some collection of orders (all orders in the example), we can use `aggregate` with `Sum` .

What if we want to summarize the totals based on product categories? 

We can do this by constructing the aggregates dynamically

Let's say our model has a category field for specifying which 

```python
from django.db import models
from django.db.models import Case, When, Sum

class Order(models.Model):
    total_price = models.DecimalField(max_digits=10, decimal_places=2)
    category = models.CharField(max_length=50)
```

Now, if we want to print out order totals by category, we need to create a aggregation query which sums the values only for the same category. If we know all the categories beforehand, we can do this by defining the aggregation queries by hand with Case and When.

```python
aggregated_data = Order.objects.aggregate(electornics=Sum(
        Case(
            When(category="Electronics", then='total_price'),
            default=0,
            output_field=models.DecimalField()
        )
    )
```

But we don't want to do this manually for each category. And what if we don't know the categories we want to include beforehand?

To achieve this we will need to build the queries dynamically, meaning, the resulting query will change based on which categories we want to be included. 

We can do this by passing the Django `aggregate` method an unpacked dictionary which contains the aggregation expressions for different categories. This is equivalent to passing the dictionary values as parameters to the `aggregate` method.

```python
arg_dict = {
	"arg1": 1,
	"arg2": 2
}
# unpack the dictionary as keyword arguments using ** operator
# same as some_func(arg1=1, arg2=2)
some_func(**arg_dict)

```

In our case, we can define a dictionary with the category names as keys and the aggregation expessions as values.

```python
aggregation_expression = {
	"electronics": Sum(
            Case(When(category="Electronics", then='total_price'),
                 default=0,
                 output_field=models.DecimalField())),
    "books": Sum(
            Case(When(category="Books", then='total_price'),
                 default=0,
                 output_field=models.DecimalField()))
}

aggregated_data = Order.objects.aggregate(**aggregation_expression)
```

As the final step, let's create a function to generate the dictionary from a list of categories. In a real application we could use this function in an API endpoint and pass a request parameter to the function.  

```python
from django.db import models
from django.db.models import Case, When, Sum

from .models import Order


def totals_by_category(categories):
    # Construct the dynamic aggregation expression and dictionary
    aggregation_expression = {}

    for category in categories:
        field_name = f"{category.lower()}"
        aggregation_expression[field_name] = Sum(
            Case(When(category=category, then='total_price'),
                 default=0,
                 output_field=models.DecimalField()))

    # Perform the aggregation
    aggregated_data = Order.objects.aggregate(**aggregation_expression)
    return aggregated_data



```

If we call `totals_by_category` with a list of categories like `["Electronics", "Books"]`, we will get a dictionary with the results as a return value.

```bash
{
    'electronics': 500.00,
    'books': 200.00
}

```

And there we have it! I hope you enjoyed this article. If you have any questions or feedback please don't hesitate to contact me! 


