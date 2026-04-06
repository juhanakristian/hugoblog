---
title: "Dark mode using prefers-color-scheme rule and CSS variables"
draft: false
date: "2020-05-14T09:33:01+00:00"
author: juhana.jauhiainen
tags: ["css", "design", "usability"]
description: "In this article I'll show you how to implement dark mode in your blog or site using CSS variables and how the toggle it automatically based on the users preferred color scheme."
---

In this article I'll show you how to implement dark mode in your blog or site using [CSS variables](https://developer.mozilla.org/en-US/docs/Web/CSS/--*) and how the toggle it automatically based on the users [preferred color scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme).

CSS variables are a new way to make CSS more maintainable by defining reusable variables which are defined in one place and can be referenced in multiple other places.

For our dark mode example we'll be changing the sites background and text colors when the user has set dark mode as preference at [system level](https://support.apple.com/en-us/HT208976).

So let's define our variables.

```css
:root {
  --background-color: #1b1b1b;
  --text-color: #d0d0d0;
}
```

Here we've defined two variables `--background-color`and `--text-color` which we'll be using in our CSS definitions. They are added to the `:root` score so they are available globally.

But first we have to add a `@media`queries with the `prefers-color-scheme`rule to enable automatic color scheme selection based on the users settings.

```css
@media (prefers-color-scheme: light) {
  :root {
    --background-color: #ffffff;
    --text-color: #000000;
  }
}

@media (prefers-color-scheme: dark) {
  :root {
    --background-color: #1b1b1b;
    --text-color: #d0d0d0;
  }
}
```

Now we're ready to use our CSS variables in our definitions.

```css
body {
  background-color: var(--background-color);
  color: var(--text-color);
}
```

Now the user should get the color scheme they prefer based on their system settings.

You can see this code in action on my personal website at [juhanajauhiainen.com](http://juhanajauhiainen.com)

If you liked this article and want to hear more from me, you can follow me on [Twitter](https://twitter.com/juhanakristian)
