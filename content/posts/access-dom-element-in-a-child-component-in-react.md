---
title: "Access DOM element in a child component in React"
draft: false
date: "2020-08-18T06:45:01+00:00"
author: juhana.jauhiainen
tags: ["react", "ref", "dom"]
description: "Sometimes we need access a DOM element which is contained by another React component. If we try to just use ref and pass it to the child component, we get an error. This is because ref is a reserved prop name so you can't pass it to a child component. Instead we can use forwardRef when defining the child component."
---

Sometimes we need access a DOM element which is contained by another React component. If we try to just use [ref](https://reactjs.org/docs/refs-and-the-dom.html) and pass it to the child component, we get an error.

```jsx
function ChildComponent(props) {
  return <div ref={props.ref}>Hello there!</div>;
}

export default function App() {
  const childRef = React.useRef();

  return (
    <div className="App">
      <ChildComponent ref={childRef} />
    </div>
  );
}
```

![](/images/referror.png)

This is because ref is a reserved prop name so you can't pass it to a child component. Instead we can use forwardRef when defining the child component.

```jsx
const ChildComponent = React.forwardRef((props, ref) => {
  return <div ref={ref}>Hello there!</div>;
});

export default function App() {
  const childRef = React.useRef();

  useEffect(() => {
    console.log(childRef);
  }, [childRef]);

  return (
    <div className="App">
      <ChildComponent ref={childRef} />
    </div>
  );
}
```

This is pretty nice, and if you're building a component library, probably the best way to allow your users to access DOM elements of the components.

There is also another way to solve the problem. We can just use a prop name which isn't reserved!

```jsx
function ChildComponent(props) {
  return <div ref={props.innerRef}>Hello there!</div>;
}

export default function App() {
  const childRef = React.useRef();

  return (
    <div className="App">
      <ChildComponent innerRef={childRef} />
    </div>
  );
}
```

This works perfectly fine!

If you want to maintain a consistent API or you're developing a component library you probably should use [forwardRef](https://reactjs.org/docs/forwarding-refs.html) but if you're developing an app you could also just use another prop name.

---

[https://reactjs.org/docs/forwarding-refs.html](https://reactjs.org/docs/forwarding-refs.html)
