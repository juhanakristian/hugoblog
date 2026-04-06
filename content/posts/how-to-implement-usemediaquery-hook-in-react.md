---
title: "How to implement useMediaQuery hook in React"
draft: false
date: "2022-03-13T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["react", "css"]
description: "Media Queries are a CSS feature that can be used to conditionally apply selected styles on an HTML element. Some examples of media queries include checking for the width of the browser window, checking for the media type (print, screen), or checking for dark/light mode preference."
---

### What are Media Queries?

[Media Queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries) are a CSS feature that can be used to conditionally apply selected styles on an HTML element. Some examples of media queries include checking for the width of the browser window, checking for the media type (print, screen), or checking for dark/light mode preference.

The most common use case for media queries is using it to implement responsivity on a website. Checking for the width of the viewport and applying styles based on it allows us to define different styles on different devices (desktop, mobile, tablet).

The syntax for media queries is comprised of an optional _media type_ and any number of _media feature_ expressions. Media types include _all_,_screen_, and _print_. The default value for media type is _all_.

```css
.header {
  font-size: 20rem;
}

@media print {
  .header {
    font-size: 15rem;
  }
}
```

The _media type_ is followed by any number of _media feature_ expressions enclosed in parenthesis.

```css
.header {
  font-size: 20rem;
  color: pink;
}

@media (max-width: 800px) {
  .header {
    color: blue;
  }
}
```

Here, `max-width: 800px` is a _media feature_ expression which signifies that this CSS will only be applied if the width of the viewport is equal to or less than 800 pixels.

There is a number of different [media features](https://developer.mozilla.org/en-US/docs/Web/CSS/@media#media_features) we can use to apply CSS in specific situations. The most common is [width](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/width) because it is used when creating responsive and mobile-friendly websites.

### How to use Media Queries from JavaScript?

If you want to check for a media query using JavaScript, you can use the [window.matchMedia](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia) function. `matchMedia` takes a single parameter, the query string you want to check for, and returns a [MediaQueryList](https://developer.mozilla.org/en-US/docs/Web/API/MediaQueryList) object. The MediaQuery object can be used to check for a match or to attach a `change` event listener. The `change` listener is called every time the result of the media query changes.

```javascript
const result = window.matchMedia("(max-width: 800px)");
if (result.matches) {
  // do something
}

result.addEventListener("change", (event) => {
  if (event.matches) {
    // do something
  }
});
```

### Custom React hook for Media Queries

`matcMedia` makes it possible to implement a React Hook we can use to check for Media Query matches and change your application UI or behavior based on the results.

First, let's define the API for our hook. With TypeScript, our hooks type definition would look like this.

```typescript
type useMediaQuery = (query: string) => boolean;
```

Our hook will take the query string as a parameter and return a boolean. Next, we'll need to add a `React.useEffect` call which calls `matchMedia` and adds an event listener for `change`. We also need a state variable to store the match.

```javascript
function useMediaQuery(query) {
  const [matches, setMatches] = React.useState(false);
  React.useEffect(() => {
    const matchQueryList = window.matchMedia(query);
    function handleChange(e) {
      setMatches(e.matches);
    }
    matchQueryList.addEventListener("change", handleChange);
  }, [query]);

  return matches;
}
```

Great 🥳 This already works but we have one more important thing to add.. cleanup for the event handler. `React.useEffect` function can return a function used for _cleanup_. It's commonly used to unregister event handlers or unsubscribing from external data sources.

Let's add a cleanup function to our `useEffect`

```js
function useMediaQuery(query) {
  const [matches, setMatches] = React.useState(false);

  React.useEffect(() => {
    const matchQueryList = window.matchMedia(query);
    function handleChange(e) {
      setMatches(e.matches);
    }
    matchQueryList.addEventListener("change", handleChange);

    return () => {
      matchQueryList.removeEventListener("change", handleChange);
    };
  }, [query]);

  return matches;
}
```

Now our `useMediaQuery` hook is finished. Here's how you would use it.

```js
function SomeComponent() {
  const isMobile = useMediaQuery("min-width: 768px)");

  return <h1>Browsing with {isMobile ? "phone" : "desktop"}</h1>;
}
```

### Links

[MDN on window.matchMedia](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia)  
[MDN on MediaQueryList](https://developer.mozilla.org/en-US/docs/Web/API/MediaQueryListEvent)
