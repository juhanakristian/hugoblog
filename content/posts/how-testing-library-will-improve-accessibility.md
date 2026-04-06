---
title: 'How using Testing Library will help you improve the accessibility of your application'
draft: false
date: '2021-02-23T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["testing-library", "accessibility", "javascript"]
description: "Testing Library is a JavaScript testing framework which focuses on testing the way the application is used."
---

[Testing Library](https://testing-library.com) is a JavaScript testing framework that focuses on testing the way the application is used. Testing Library will also help you avoid testing implementation details and make your tests more maintainable. Testing the way the application is used will give us the confidence that the application is working as intended.

What's also nice about testing-library, is that its recommended queries are designed to work well on accessible elements. This means using Testing Library will also reveal accessibility problems in your application. 

In this article, we'll go through a few examples where Testing Library will reveal accessibility problems in your application.

<div className="bg-yellow-200 text-black font-thin p-6 rounded-md flex gap-4 shadow-md">
<div>👉</div>
<div>While this article uses React components as examples, testing-library supports many other frameworks and vanilla JS</div>
</div>

---
## Button 

The first example we'll look at is implementing a button. Here's a naive way of creating a button component.

```js

function Button(props) {
    return (<div className="btn" onClick={props.onClick}>{props.children}</div>);
}

test("button exists", () => {
    render(<Button>Click Me</Button>);
    expect(screen.getByRole("button", {name: /click me/i})).toBeInDocument();
})

```

Here we have a button implemented using a `div` element and when you try to get it using a `getByRole` query in your tests you'll quickly realize it doesn't work.

The test will fail because the query can't find the button. Now, you might just use `getByText` and call it a day. But the problem is, screen readers won't recognize the div-button as a clickable element and the users who depend on them won't be able to use your application at all!

The best way to fix this is to just use the `button` element instead of a `div` element. This will ensure it will be visible to assistive technologies.

If for some reason you still need to use `div` you can add the `role` attribute to the element.

```html
<div className="btn" role="button" onClick={props.onClick}>{props.children}</div>
```

Implementing buttons using divs might seems like a far fetched example but well, it happens 😅

---
## Modal

As the next example, we will look at implementing a modal. Here's a simple modal implementation.

```js
function Modal({open, title, children}) {
  return (
    <div className="backdrop" style={{display: open ? "block" : "none"}}>
      <div className="modal">
        <h3>{title}</h3>
        <div>
          {children}
        <div>
      </div>
    </div>
  )
}

test("modal has correct title", () => {
    render(<Modal title="Modal title">modal content</Modal>);
    expect(screen.getByRole("dialog", {name: /modal content/i})).toBeInDocument();
})
```

This test will fail to find the dialog, and from the perspective of assistive technologies, the modal might as well not exist. You could get around this issue by querying the div element with the class `modal` but then you would be testing implementation details. What happens when someone decides to change the class name?

Instead, you should make the modal accessible by adding `role`, `aria-modal`, and `aria-labelledby` attributes.

`role` identifies the element as a dialog  
`aria-modal` indicates that the elements under the dialog can't be interacted while the dialog is open  
`aria-labelledby` gives the dialog an accessible name by referencing the element which gives the dialog its title  

```js
<div className="modal"
      role="dialog"
      aria-modal="true"
      aria-labelledby="dialog-title">
  <h3 id="dialog-title">{title}</h3>
  <div>
    {children}
  <div>
</div>
```

---
## Reach UI

Instead of implementing controls, modals, etc. completely from scratch, I recommend using [Reach UI](https://reach.tech/). It is an accessible foundation for your own components and makes creating accessible design systems easy.

## Further reading

[Testing Library](https://testing-library.com/)  
[Commong mistakes testing](https://kentcdodds.com/blog/common-testing-mistakes) by Kent C. Dodds  
[Common mistakes with React Testing Library](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)  by Kent C. Dodds  

<!-- <span>Photo by <a href="https://unsplash.com/@untodesign_?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Daniel Ali</a> on <a href="https://unsplash.com/s/photos/accessibility?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span> -->