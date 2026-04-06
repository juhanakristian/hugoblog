---
title: "Generating forms from JSON schema in React applications"
date: "2020-01-19T20:19:10+00:00"
draft: false
permalink: /generating-forms-from-json-schema-in-react-applications
author: juhana.jauhiainen
excerpt: ""
type: post
category:
  - Uncategorized
tags: []
post_format: []
description: "
Automatically generating UI from a schema document can be a powerful prototyping tool or can even be used to build simple apps if the requirements aren't too complex.

One awesome library for generating UIs from json-schemas is react-jsonschema-form. react-jsonschema-from can be used to automatically generate forms in a React application from a JSON schema document.
"
---

Automatically generating UI from a schema document can be a powerful prototyping tool or can even be used to build simple apps if the requirements aren't too complex.

One awesome library for generating UIs from [json-schemas](https://json-schema.org/) is [react-jsonschema-form](https://github.com/rjsf-team/react-jsonschema-form). react-jsonschema-from can be used to automatically generate forms in a React application from a JSON schema document.

I found the library easy to use and it was perfect for my use case of a form input app which could later be expanded with additional forms with minimal coding.

For example a simple JSON schema with firstname and lastname properties can be used to generate a full form with title, description, input fields and a submit button.

```json
{
  "title": "A registration form",
  "description": "A simple form example.",
  "type": "object",
  "required": ["firstName", "lastName"],
  "properties": {
    "firstName": {
      "type": "string",
      "title": "First name",
      "default": "Chuck"
    },
    "lastName": {
      "type": "string",
      "title": "Last name"
    }
  }
}
```

![](/images/reactjsonform.png)

By default the form is styled using Bootstrap but the inputs can be styled as by using a special UI schema. You can even use a UI schema to define the widgets for the inputs and also define some additional css classes which will be added to the input elements.

```json
{
  "firstName": {
    "classNames": "additional-css-class"
  }
}
```

Customization can be taken even further by creating custom React components for fields. There is also a Material UI port of the library called [react-jsonschema-form-material-ui](https://github.com/vip-git/react-jsonschema-form-material-ui)
