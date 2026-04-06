---
title: 'Python class and instance variables'
date: '2020-03-11T18:18:03+00:00'
draft: false
permalink: /python-class-and-instance-variables
author: juhana.jauhiainen
tags: ['python']
description: "Confusing class variables and instance variables in Python is a pretty common mistake. Or.. atleast I've made it more than once.
Python doesn't use static keyword but instead the location the variables are defined in to differentiate between the two which will seem pretty foreign to people coming from other object oriented languages."
---

Confusing class variables and instance variables in Python is a pretty common mistake. Or.. atleast I've made it more than once.
Python doesn't use `static` keyword but instead the location the variables are defined in to differentiate between the two which will seem pretty foreign to people coming from other object oriented languages.

Here's an example of how you would declare class variables in Python
```python
class Foo:
    bar = True

instance = Foo()
```

`bar` is now accessible by `Foo.bar`, `self.bar` inside the methods of class `Foo`
and with `instance.bar` using a instance of the class.

Now here's an example how to declare a instance variable in Python.

```python
class Foo:
    def __init__(self):
        self.bar = True

instance = Foo()
```

`bar` is now accessible with `self.bar` inside class `Foo` methods and `instance.bar`
using a instance of the class.

So what's the difference between these two? In addition to the already mentioned
access rules, class variables are shared between all instances of the class while instance variables
are unique to the instance.

If this was not the intention, it can lead to hard to trace bugs.

Coming from other object oriented programming languages you might  
think you need to declare the variable first and then initialize it in `__init__`.

```python
class Foo:
    bar = False

    def __init__(self, set_bar):
        self.bar = set_bar
```

This is however not necessary and while it probably won't cause issues, because
the instance variable hides the class variable, it's not very Pythonic.


