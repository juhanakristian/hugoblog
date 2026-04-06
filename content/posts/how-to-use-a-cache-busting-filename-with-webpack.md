---
title: "How to use a cache busting filename with webpack"
draft: false
date: "2021-07-17T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["webpack", "javascript", "caching"]
description: "If you're using a custom webpack config, you'll want to make sure the filename has a cache busting suffix."
---

If you're using a custom webpack config, instead of a template or [Create React App](https://create-react-app.dev/), you'll want to make sure the filename has a cache busting suffix. Cache busting means making sure the users browser loads the newest version of our app, if it has been updated, instead of using a cached version. 

A naive webpack setup would just use a single output file with a fixed name.

```js
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: './src/index.js',
    plugins: [
      new HtmlWebpackPlugin(),
    ],
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist'),
    },
  }
```

Here, we are using [HtmlWebpackPlugin](https://webpack.js.org/plugins/html-webpack-plugin/) to manage the creation of our **index.html** file and it will handle inserting a `<script>` tag into the **index.html** with our bundle filename. For now we are bundling all of our JavaScript into single file called **bundle.js**.

Using a fixed filename for the bundle is problematic because the browser will cache the file and use the cached version even when the file has changed. What we want to do is to add a **suffix** to our filename and change it everytime the code changes.

We can achieve this by using webpack filename [substitutions](https://webpack.js.org/configuration/output/#outputfilename). Let's change the output filename to include `[name]` and `[contenthash]`.

```js
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: './src/index.js',
    plugins: [
      new HtmlWebpackPlugin(),
    ],
    output: {
      filename: '[name].[contenthash].js',
      path: path.resolve(__dirname, 'dist'),
    },
  }
```

Now the output filename will include a hash of the output content so the filename will be different if the output bundle has changed 🥳

### Further reading

[webpack docs on caching](https://webpack.js.org/guides/caching/)

