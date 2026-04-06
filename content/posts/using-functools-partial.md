---
title: "Using functools.partial"
date: "2020-02-23T20:28:34+00:00"
draft: false
permalink: /using-functools-partial
author: juhana.jauhiainen
excerpt: ""
type: post
category:
  - Uncategorized
tags: ["python"]
post_format: []
description: "
I’ve gotten used to a functional programming style in JavaScript and I’m finding I want to use same kind of paradigms in Python.

Builtin methods like map() and filter() already help but the functools module provides additional ways to do functional programming.

Let’s look at what the functools.partial function can do.
"
---

I’ve gotten used to a functional programming style in JavaScript and I’m finding I want to use same kind of paradigms in Python.

Builtin methods like **map()** and **filter()** already help but the **functools** module provides additional ways to do functional programming.

Let’s look at what the **functools.partial** function can do.

**partial()** can be take a function of many parameters and make it a function of few paramters. This is very useful if you have to call the same function  
multiple times with almost the same parameters.

Here’s an example

```python
from functools import partial

def greet(greeting, person):
    print(f'{greeting} {person}!')

say_hello = partial(greet, 'Hello')

say_hello('John')
# prints 'Heloo John!'
```

You can also use keyword arguments with partial to define which parameters are defined.

```python
greet_john = partial(greet, name='John')

greet_john(greeting='Howdy')
# prints 'Howdy John!'

greet_john(greeting='Hi')
# prints 'Hi John!'
```

Pretty neat! There are some other interesting functional programming tools in functools module which I’m going to explore in the future.
