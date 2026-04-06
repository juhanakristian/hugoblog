---
title: "Customize Django admin with list_display property"
draft: false
date: "2020-07-23T23:32:01+00:00"
author: juhana.jauhiainen
tags: ["django", "python"]
description: "One of the great \"batteries included\" features Django has, is the automatically generated admin panel. It provides a simple UI for creating, editing and deleting data defined with the Django ORM. In this article we are going to enable the admin user interface for a simple model and customize it from a simple list view to a more user friendly table like interface."
---

One of the great "batteries included" features Django has, is the automatically generated admin panel. It provides a simple UI for creating, editing and deleting data defined with the Django ORM. In this article we are going to enable the admin user interface for a simple model and customize it from a simple list view to a more user friendly table like interface.

Lets say we have a simple model `Item` which has two fields `name`and `price`.

```python
class Item(models.Model):
	name = models.CharField(max_length=50)
	price = models.DecimalField(max_digits=5, decimal_places=2)
```

Add the model to the admin page we only have to register it in the [apps`admin.py`](http://appsadmin.py) file.

```python
from django.contrib import admin
from .models import Item

admin.site.register(Item)
```

Now when we run the app and open a browser to `[http://localhost:8000/admin](http://localhost:8000/adminwe)` and add a `Item`with the name `Pizza` using the admin panel.

![](/images/django_admin_0.png)

By default the Django admin site displays model objects as a simple list with the string representation of the model object as title. Our model class doesn't provide a `__str__` method so Django uses the models name as the title. We can fix this by adding a `__str__`method to our `Item`model.

```python

class Item(models.Model):
	name = models.CharField(max_length=50)
	price = models.DecimalField(max_digits=5, decimal_places=2)

	def __str__(self):
		return self.name
```

Now the admin interface looks much better.

![](/images/django_admin_1.png)

This is already nice but it would be easier to browse the existing `Item`values if they were displayed as a table with values instead of a plain list.

We can customize the display by creating a custom admin model class and setting the value of `list_display`property. Let's add a `ItemAdmin`model to our `admin.py`file

```python
from django.contrib import admin
from .models import Item

class ItemAdmin(admin.ModelAdmin):
	list_display = ("name", "price",)

admin.site.register(Item, ItemAdmin)
```

Now we get a table like view of the existing `Item` objects.

![](/images/django_admin_2.png)

`ModelAdmin`also allows us to create dynamic fields by declaring methods in `ItemAdmin`and adding them to the `list_display`property.

```python
from django.contrib import admin
from .models import Item

from decimal import Decimal

class ItemAdmin(admin.ModelAdmin):
    list_display = ("name", "price", "vat")

    def vat(self, obj: Item) -> str:
        return f"{(obj.price * Decimal(0.05)):.2f}$"

admin.site.register(Item, ItemAdmin)
```

Now we will get an extra field `vat` in our admin panel.

![](/images/django_admin_3.png)

Another easy improvement to our admin view is allowing editing in the table view. Let's first add a boolean field `is_available` to our model.

```python
from django.db import models

class Item(models.Model):
    name = models.CharField(max_length=50)
    price = models.DecimalField(max_digits=5, decimal_places=2)
    is_available = models.BooleanField(default=True)

    def __str__(self):
        return self.name
```

We can allow editing of a field by adding it to `list_editable`tuple property in it's model admin class.

```python
from django.contrib import admin
from .models import Item

from decimal import Decimal

class ItemAdmin(admin.ModelAdmin):
    list_display = ("name", "price", "vat", "is_available")
    list_editable = ("is_available",)

    def vat(self, obj: Item) -> str:
        return f"{(obj.price * Decimal(0.05)):.2f}$"

admin.site.register(Item, ItemAdmin)
```

Now we can set the `is_available`value from the list view using a checkbox, without opening the detailed view.

![](/images/django_admin_4.png)

For further reading I recommend going trough the Django documentation on [ModelAdmin.](https://docs.djangoproject.com/en/3.0/ref/contrib/admin/#django.contrib.admin.ModelAdmin)

The example project with full source code is available at my [github](https://github.com/juhanakristian/adminexample)
