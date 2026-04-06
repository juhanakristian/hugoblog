---
title: "What is React Strict Mode?"
draft: false
date: "2020-09-26T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["react", "frontend", "debugging"]
description: "React Strict Mode is a tool, which comes with React, for detecting possible issues and problems in your application. Currently (Sept. 2020) Strict Mode detects if you have unsafe lifecycle methods, usage of legacy string ref API, usage of findDOMNode, detecting unexpected side effects or detecting usage of legacy context API."
---

React Strict Mode is a tool, which comes with React, for detecting possible issues and problems in your application. Currently (Sept. 2020) Strict Mode detects if you have [unsafe lifecycle methods](https://reactjs.org/docs/strict-mode.html#identifying-unsafe-lifecycles), [usage of legacy string ref API](https://reactjs.org/docs/strict-mode.html#warning-about-legacy-string-ref-api-usage), [usage of findDOMNode](https://reactjs.org/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage), [detecting unexpected side effects](https://reactjs.org/docs/strict-mode.html#detecting-unexpected-side-effects) or [detecting usage of legacy context API](https://reactjs.org/docs/strict-mode.html#detecting-legacy-context-api).

So basically, using Strict Mode will help you detect if your app or libraries are using React APIs which are deprecated, unsafe in async code or have problems which might cause bugs. **Strict Mode warnings are only displayed in development**, so you don't have to worry about them showing up in production. In the future, Strict Mode will likely add other warnings so even if you don't have any now, it's a good idea to keep using it.

Tools like create-create-app will add Strict Mode to your app by default, but adding it to your app later is also super easy. All you need to do is wrap your app or a portion of your app with `React.StrictMode`component.

```jsx
import React from "react";
import ReactDOM from "react-dom";

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById("root")
);
```

Now you'll see the possible warnings in the browser developer tools console when you run your app.

-Juhana
