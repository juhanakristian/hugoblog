---
title: "Create a React Hook to show browser online status"
draft: false
date: "2020-05-31T09:33:01+00:00"
author: juhana.jauhiainen
tags: ["react", "hooks"]
description: "React Hooks are a way of adding logic to your functional components in an simple and elegant way. In addition to the standard hooks like useState or useEffect you can also create your own hooks if find you need the same logic in multiple components."
---

React Hooks are a way of adding logic to your functional components in an simple and elegant way. In addition to the [standard hooks](https://reactjs.org/docs/hooks-reference.html) like `useState` or `useEffect` you can also create your own hooks if find you need the same logic in multiple components.

A [custom hook](https://reactjs.org/docs/hooks-custom.html) is nothing more than a JavaScript function which starts with "use" and that may call other hooks. In this article I will show you how to create a simple hook for showing the network status of the users browser.

To access the browsers online we'll use the [NavigatorOnLine API](https://developer.mozilla.org/en-US/docs/Web/API/NavigatorOnLine/Online_and_offline_events) to read the status and tu subscribe to the `online` and `offline` events.

Let's start by creating a Hook which reads the current online status.

```jsx
function useOnlineStatus() {
  return window.navigator.onLine
}
```

The hook is a simple function which returns the value of `window.navigator.onLine`. You can use it by calling `useOnlineStatus`in your component.

```jsx
function Component() {
  const isOnline = useOnlineStatus()
  const text = isOnline ? "online" : "offline"
  return <p>{`Your browser is ${text}`}</p>
}
```

This already kinda works but the status won't update if the browser goes offline after the component has been mounted. We'll need to add some event listeners and a local state variable to achieve that.

```jsx
function useOnlineStatus() {
  const [online, setOnline] = useState(window.navigator.onLine)

  useEffect(() => {
    window.addEventListener("online", () => setOnline(true))
    window.addEventListener("offline", () => setOnline(false))
  }, [])

  return online
}
```

Now we get the updated state when the browser goes offline or online. But we are not done yet. Our hook function is not doing proper cleanup for the event listeners, which will result in memory leaks if not fixed.

We need to return a cleanup function from `useEffect`, which will remove the event listeners when the component using the hook is unmounted. To achieve this we also need to change our event handler functions from [arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) to regular named functions.

```jsx
function useOnlineStatus() {
  const [online, setOnline] = useState(window.navigator.onLine)

  useEffect(() => {
    function handleOnline() {
      setOnline(true)
    }

    function handleOffline() {
      setOnline(false)
    }

    window.addEventListener("online", handleOnline)
    window.addEventListener("offline", handleOffline)

    return () => {
      window.removeEventListener("online", handleOnline)
      window.removeEventListener("offline", handleOffline)
    }
  }, [])

  return online
}
```

Now we are properly cleaning up and the hook is ready to be used!

You can check a full code example at [codesandbox.io](https://codesandbox.io/s/dawn-moon-4487x?file=/src/App.js)
