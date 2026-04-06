---
title: 'The very basics of React Hooks'
draft: false
date: '2020-04-27T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["javascript", "react", "hooks"]
description: "Hooks are an React API that were introduced in React 16.8 a little more than a year ago (february 2019).
They let you use state in your React components without writing classes."
---
[Hooks](https://reactjs.org/docs/hooks-intro.html) are an React API that were introduced in [React 16.8](https://reactjs.org/blog/2019/02/06/react-v16.8.0.html) a little more than a year ago (february 2019).
They let you use [state](https://reactjs.org/docs/state-and-lifecycle.html) in your React [components](https://reactjs.org/docs/components-and-props.html) without writing classes.

In this post I'll talk about the `useState` hook and after reading it you'll have a basic idea how to use it to add state to your functional components.

Let's start with a simple functional component.
```jsx

import React from "react";
function Cat() {
  return (
    <span>
      🐱
    </span>
  );
}

```

Here we are simply rendering a cat emoji inside a humble span element.

Now let's add some state using the `useState` hook!

We want to track how many times the cat has been petted and display a different (😻) emoji based on the `petted` count.

```jsx
import React, { useState } from "react";

function Cat() {
  const [petted, setPetted] = useState(0);
  return (
    <span>
      🐱
    </span>
  );
}
```

First I've imported the `useState` hook. 

Then I use it to create a local state variable `petted` and a function, `setPetted` which can be used to update the state. 

So `useState` takes a single argument, the initial state, and returns a array of two elements, the state variable and a function for changing the state. The state created with `useState` will be preserved between [renders](https://reactjs.org/docs/rendering-elements.html) and if it changes, a new render call will be triggered.

Now we are going to see how we can use `setPetted` to update our state and change the emoji based on the value of `petted`.

```jsx
import React, { useState } from "react";

function Cat() {
  const [petted, setPetted] = useState(0);
  return (
    <span onClick={() => setPetted(petted + 1)}>
      🐱
    </span>
  );
}
```

Now we have added a onClick [event handler](https://reactjs.org/docs/handling-events.html) which is bound to an [arrow function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) that calls `setPetted` to change the value of the state variable `petted`.

So are we done?

Well not yet. We still have to use the value of `petted` to displaying the correct emoji.

```jsx
import React, { useState } from "react";

function Cat() {
  const [petted, setPetted] = useState(0);
  const emoji = petted > 5 ? "😻" : "🐱";
  return (
    <span onClick={() => setPetted(petted + 1)}>
      {emoji}
    </span>
  );
}
```

Here we are using a [ternary operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) to select which of the two emojis we assign to `emoji` variable. 

Finally we use the `emoji` variable inside the span element to display the cat emoji.

[Here's](https://codesandbox.io/s/quirky-edison-hgw2y?file=/src/App.js) a codesanbox with the code and some CSS improvements