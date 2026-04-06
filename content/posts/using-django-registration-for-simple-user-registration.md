---
title: "Using django-registration for simple user registration"
date: "2019-07-05T21:10:01+00:00"
draft: false
permalink: /using-django-registration-for-simple-user-registration
author: juhana.jauhiainen
excerpt: ""
type: post
category:
  - Uncategorized
tags: ["django", "python"]
post_format: []
description: "
Recently I had to setup a web app with simple one step user registration. After some searching I came accross a Django library called django-registration which seemed like the best way to implement user registration. It features single step and two step registration flows, some simple views and simple installation and configuration. In this blog post I’m going to go over the steps needed to implement user registration in a Django app.
"
---

Recently I had to setup a web app with simple one step user registration. After some searching I came accross a Django library called django-registration which seemed like the best way to implement user registration. It features single step and two step registration flows, some simple views and simple installation and configuration. In this blog post I’m going to go over the steps needed to implement user registration in a Django app.

I’m assuming a Django project has already been setup. It’s propably a good idea to create a new app for the user registration and authentication code. In the code examples I’m going to assume there’s an app called authentication which contains the user registration code.

First step is installing django-registration.

```shell
pip install django-registration
```

We’re going to be using a custom Django User model so we can later add fields to our User model without too much of a hassle. This isn’t strictly required so feel free to skip this step if you want.

Defining a custom User model in Django is done by subclassing AbstractUser. I’m going to be using email as username so I’ve modified the email field to be unique, set **username** to **None** and set **USERNAME_FIELD** to **email**.

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    email = models.EmailField(unique=True)
    username = None
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = []
```

Next we will define our registration form which uses our custom User model.

```python
from django_registration.forms import RegistrationForm
from .models import User

class UserForm(RegistrationForm):
    class Meta(RegistrationForm.Meta):
        model = User

```

And define update **urls.py** with the registration url.

```python
from django.contrib import admin
from django.urls import path, include

from django_registration.backends.one_step.views import RegistrationView

from authentication.forms import UserForm

urlpatterns = [
    path('admin/', admin.site.urls),
    path('register/',
         RegistrationView.as_view(
             form_class=UserForm
         ),
         name='django_registration_register',
    ),
    path('', include('django_registration.backends.one_step.urls')),
    path('', include('django.contrib.auth.urls')),
]

```

In addition to the registration url we’re including the one step registration urls from djano-registration and the usual auth urls from Django.

Finally we need to create a template for rendering the registration form.

```html
<!DOCTYPE html>
<html>
  <meta charset="utf-8" />
  <head>
    <title></title>
  </head>
  <body>
    <form method="post">
      {% csrf_token %} {{form}}
      <input type="submit" value="Submit" />
    </form>
  </body>
</html>
```

This template must be saved as **django_registration/registration_form.html** in order for it to be found by RegistrationView.

And that’s really it. Now you should be able to register a new user using just email and password from <http://localhost:8000/register>

A full example of using django registration can be found from my [github](https://github.com/juhanakristian/DjangoUserRegistrationExample)
