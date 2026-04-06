---
title: "Use Model Manager to filter queryset by default"
draft: false
date: "2022-01-18T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["django", "python", "webdev"]
description: "How can you filter a queryset by default so that no matter where you use the model, the filter will be applied?"
---

How can you filter a queryset by default so that no matter where you use the model, the filter will be applied?

This can be achieved by creating a custom [Manager](https://docs.djangoproject.com/en/4.0/topics/db/managers/) and overriding its `get_queryset` method.

For example, if we have a **soft delete** feature in our app implemented by adding a `deleted` column, we can force only returing non-deleted objects by creating a custom model manager which filters for `deleted=false`

```python
class BookManager(models.Manager):
	def get_queryset(self):
		return super().get_queryset().filter(deleted=False)

class Book(models.Model):
	# ...
    deleted = models.BooleanField(default=False)
	objects = BookManager()
```

Now everytime you query Book objects in your app with `Book.objects` , you will only receive books that haven't been deleted.

### Links

[Model managers](https://docs.djangoproject.com/en/4.0/topics/db/managers/)
