---
title: "Memoization in Python"
draft: false
date: "2020-04-17T09:33:01+00:00"
permalink: /formatting-json-by-building-a-parse-tree
author: juhana.jauhiainen
tags: ["optimization", "python"]
description: "Memoization is a optimization technique frequently used in functional
programming. It means storing the result of a computationally intensive function
call and returing the cached result when the function is called with the same arguments."
---

Memoization is a optimization technique frequently used in functional
programming. It means storing the result of a computationally intensive function
call and returing the cached result when the function is called with the same arguments.

Python 3.2 introduced the `functools.lru_cache` decorator which enables
memoization of function results in Python.

Using `lru_cache` is simple as we can see from a example taken from the [Python documentation](https://docs.python.org/3.8/library/functools.html)

```python
@lru_cache
def count_vowels(sentence):
    sentence = sentence.casefold()
    return sum(sentence.count(vowel) for vowel in 'aeiou')
```

When calling for ex. `count_vowels('Count vowels in this sentence')` the result
is cached and when you call the function with the same argument the cached
result is returned instead of counting the vowels again. This can be a huge performance boost when dealing with big datasets with potentially similar data.

`lru_cache` has two optional parameters `maxsize` and `typed`. `maxsize` can be
used to limit the number of cached results. If `typed` is set to True the
arguments of different types will be cached separately.

In Python 3.8 another decorator
`functools.cached_property` was introduced for memoization of class method values.

Using `cached_property` is pretty straightforward.

```python
class Example:
    def __init__(self, some_numbers):
	    self._data = some_numbers

    @cached_property
    def mean(self):
	    return statistics.mean(self._data)
```

The result of `mean` is now cached and calculated only once on the first time
the method is called.

To use this optimization technique effectively it's essential to check the cache hits and misses.
Without checking how many of the function calls actually use the cache one can't really be sure if any performance has been gained.
Luckily Python provides us with the `cached_info()` function that returns a tuple showing `hits`, `misses`, `maxsize`and `currsize`.

```python
@lru_cache(maxsize=32)
def feet_to_meters(feet):
    return 0.3048 * feet

>>> list(map(feet_to_meters, [10, 20, 100, 10]))
[3.048, 6.096, 30.48, 3.048]
>>> feet_to_meters.cache_info()
CacheInfo(hits=1, misses=3, maxsize=32, currsize=3)
```

With these statistics it's much easier to know it the memoization has been an effective optimization.
