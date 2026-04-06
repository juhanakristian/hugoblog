---
title: 'JSON diffing'
date: '2019-09-08T09:33:01+00:00'
status: hidden
permalink: /formatting-json-by-building-a-parse-tree
author: juhana.jauhiainen
excerpt: ''
type: post
id: 28
category:
    - Uncategorized
tags: []
post_format: []
---
Some time ago a client asked if I knew a tool for formatting and comparing JSON data. The use case he was after was comparing JSON data returned by an API to the expected output.

I’d been on the look for a similar tool myself mainly for formatting and inspecting sizeable JSON data dumps. A number of online tools exist for formatting JSON but most of them either were serving ads or had a horribly cluttered interface.

So of course I decided to roll my own 🙂

The app (yes it’s a app and not a website) is called Jiff and I’m developing it using Electron and React. Currently the formatting is done by tokenizing the JSON, building a parse tree and then rendering it. For syntax highlighting I plan to use [highlight.js](https://highlightjs.org/) and for diffing a wonderful library called [jsondiffpatch](https://github.com/benjamine/jsondiffpatch)

Repo is found at [github.com/brandfilt/jiff-app](https://github.com/brandfilt/jiff-app)

