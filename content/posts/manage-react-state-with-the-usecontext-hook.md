---
title: "Manage React state with the useContext hook"
draft: false
date: "2020-10-14T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["react", "context", "state"]
description: "React projects have many options for managing state. While libraries like redux and mobx are a popular choice, React also has it's own API for managing state. The Context API is useful when you have state which is accessed in multiple places in your app and you want to avoid prop drilling..."
---

**DISCLAIMER**: This post is based on state management ideas presented by Kent C. Dodd’s in [Application state management with React](https://kentcdodds.com/blog/application-state-management-with-react). If you haven't read it, I encourage you to do so.

React projects have many options for managing state. While libraries like `redux` and `mobx` are a popular choice, React also has it's own API for managing state. The Context API is useful when you have state which is accessed in multiple places in your app and you want to avoid [prop drilling](https://kentcdodds.com/blog/prop-drilling).

The Context API consist of `createContext`, used for creating the context object and `useContext`, a hook for accessing the Context.

The [useContext hook](https://reactjs.org/docs/hooks-reference.html#usecontext) takes a Context object and returns the current context value for that object. The current context value comes from a [Provider](https://reactjs.org/docs/context.html#contextprovider), which is a React component that allows subscribing to context changes.

We can start by creating a Context which stores our state.

```jsx
import React from "react";

const LanguageContext = React.createContext();
```

`createContext` will create a Context object which will store our state.

Next we will use the value in a component with the useContext hook.

```jsx
import React from "react";

const LanguageContext = React.createContext();

function LanguageDisplay() {
  const value = React.useContext(LanguageContext);
  return <h2>{`Current language is ${value}`}</h2>;
}
```

To use this component in an app we also need to have a `LanguageContext.Provider` in our component tree. A provider is the source of the values stored in the Context for all components lower in the component hierarchy. With the provider we can define a default value for the context.

```jsx
import React from "react";

const LanguageContext = React.createContext();

function LanguageDisplay() {
  const value = React.useContext(LanguageContext);
  return <h2>{`Current language is ${value}`}</h2>;
}

export default function App() {
  return (
    <LanguageContext.Provider value="en">
      <div className="App">
        <LanguageDisplay />
      </div>
    </LanguageContext.Provider>
  );
}
```

If we want to change the value stored in the context, we can wrap our Provider and use `useState` to get a function for changing the value.

```jsx
function LanguageProvider(props) {
  const [language, setLanguage] = React.useState("en");
  const value = React.useMemo(() => [language, setLanguage], [language]);
  return <LanguageContext.Provider value={value} {...props} />;
}
```

Now we can create a component for changing the language.

```jsx
function LanguageSelect() {
  const context = React.useContext(LanguageContext);

  return (
    <select
      value={context.value}
      onChange={(event) => context.setLanguage(event.target.value)}
    >
      <option value="en">English</option>
      <option value="fi">Finnish</option>
      <option value="se">Swedish</option>
    </select>
  );
}
```

Something we can also do, is to wrap `useContext`in a custom hook so we get a nice and clean interface.

```jsx
function useLanguage() {
  const context = React.useContext(LanguageContext);
  return context;
}
```

Now we have a great set of hooks and components which provide a clean interface for managing a little global state. Finally here's the full code example with a component for changing the value in the context.

```jsx
import React from "react";

const LanguageContext = React.createContext("en");

function useLanguage() {
  const context = React.useContext(LanguageContext);
  return context;
}

function LanguageProvider(props) {
  const [language, setLanguage] = React.useState("en");
  const value = React.useMemo(() => [language, setLanguage], [language]);
  return <LanguageContext.Provider value={value} {...props} />;
}

function LanguageSelect() {
  const [language, setLanguage] = useLanguage();

  return (
    <select
      value={language}
      onChange={(event) => setLanguage(event.target.value)}
    >
      <option value="en">English</option>
      <option value="fi">Finnish</option>
      <option value="se">Swedish</option>
    </select>
  );
}

function LanguageDisplay() {
  const [language] = useLanguage();
  return <h2>{`Current language is ${language}`}</h2>;
}

export default function App() {
  return (
    <LanguageProvider>
      <div className="App">
        <LanguageSelect />
        <LanguageDisplay />
      </div>
    </LanguageProvider>
  );
}
```

You can play with the example code in this [codesandbox](https://codesandbox.io/s/nostalgic-sanderson-yqz4s?file=/src/App.js)
