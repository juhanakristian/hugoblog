---
title: "Using namedtuple to create simple data objects in Python"
draft: false
date: "2020-06-29T23:30:00+00:00"
author: juhana.jauhiainen
tags: ["python", "namedtuple"]
description: "Python standard library has a handy collection factory function called namedtuple, which can be used to assign names for each position in a tuple. Named tuples behave otherwise like a normal tuples but each element has a name which can be used for retrieving the value just like with class objects.
"
---

Python standard library has a handy collection factory function called [namedtuple](https://docs.python.org/3/library/collections.html#collections.namedtuple), which can be used to assign names for each position in a tuple. 

**Named tuples behave otherwise like a normal tuples but each element has a name which can be used for retrieving the value just like with class objects.** 

```python
from collections import namedtuple
Item = namedtuple("Item", ["name", "price", "quantity"])
```

Here we first import `namedtuple`from the `collections`module in the standard library and then create a named tuple called `Item`with the properties `name`, `price` and `quantity`.

Now we can create objects of type `Item`the same way would if `Item`were a class.

```python
ketchup = Item(name="Ketchup", price=1.00, quantity=10000)
```

We can also access the properties as you would expect.

```python
>>> ketchup.price
1.0
>>> ketchup.quantity
10000
>>> toilet_paper.name
'Ketchup'
```

All the properties we have defined for `Item`are required so if you try to create a object without `quantity` we will get an error.

```python
>>> mustard = Item(name="Mustard", price=1.00)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: __new__() missing 1 required positional argument: 'quantity'
>>>
```

We can however, pass a list of default values when creating the named tuple.

```python
Item = namedtuple("Item", ["name", "price", "quantity", "is_dairy"], defaults=[False])
```

The default values are applied from right to left, so here `is_dairy`defaults to `False`. The properties with default values hence have to be defined after the properties with required values.

Since tuples are immutable, we can't change the values by simple assignment.

```python
>>> ketchup.price = 2.00
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: can't set attribute
>>>
```

But named tuples provide us with `_replace`method which we can use to change values.

```python
>>> ketchup._replace(price=2.00)
Item(price=2.0, quantity=10000, name='Ketchup')
>>>
```

For initializing a named tuple we can also use the `_make`method which takes as its input a iterable. This means we can **initialize our named tuple from a list**, a standard tuple or any other object which implements the [Python iterator protocol](https://docs.python.org/3/c-api/iter.html)

```python
>>> pepper = Item._make(["Pepper", 1.00, 1])
>>> pepper
Item(price='Pepper', quantity=1.0, name=1)
>>>
```

The Python documentation has [an great example](https://docs.python.org/3/library/collections.html#collections.namedtuple) of using `_make`to initialize named tuples when reading data from a SQLite database or when reading a CSV file.

Named tuples can be easily **converted to dictionaries and vice versa** using the `_asdict`method and the [double star operator](https://docs.python.org/3/tutorial/controlflow.html#tut-unpacking-arguments)

```python
>>> ketchup._asdict()
{'name': 'Ketchup', 'price': 1.0, 'quantity': 10000, 'is_dairy': False}
>>>
>>> mustard_dict = {"name": "Mustard", "price": 1.00, "quantity": 10}
>>> mustard = Item(**mustard_dict)
>>> mustard
Item(price=1.0, quantity=10, name='Mustard')
>>>
```

Note that as of Python 3.8, `_asdict`returns a regular `dict` because they are now also ordered (since Python 3.7).

We can access the name of the fields and the default values programmatically using the `_fields` and `_field_defaults`properties.

```python
>>> ketchup._fields
('name', 'price', 'quantity', 'is_dairy')
>>> ketchup._field_defaults
{'is_dairy': False}
>>>
```

For more on named tuples you should check the [Python documentation.](https://docs.python.org/3.8/library/collections.html#collections.namedtuple)

