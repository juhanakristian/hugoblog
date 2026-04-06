---
title: "Introduction to relative CSS units"
draft: false
date: "2020-06-22T23:38:00+00:00"
author: juhana.jauhiainen
tags: ["css"]
description: "With CSS you can use absolute units like the familiar px, pt or relative units like em and rem . Absolute units are quite simple to use and rarely cause confusion. Relative units however can twist your brain if your not familiar with how they work."
---

With CSS you can use absolute units like the familiar `px`, `pt` or relative units like `em` and `rem` . Absolute units are quite simple to use and rarely cause confusion. Relative units however can twist your brain if your not familiar with how they work.

We are going to go through some of the most common relative units with simple examples that hopefully help you understand how they work and how to use them.

First there's `em`. It's a relative unit which defines the size of a element relative to the font size of the element it's defined on. So, for example setting element padding as `2em`means it's padding will be twice the font size. If the font size is 10px there will be 20px padding inside the element.

The font size of course can be inherited from a parent element or set with the elements CSS. If you don't have any font-size definitions in your CSS, the browsers default value will be used.

```css
.box {
  font-size: 10px;
  padding: 2em;
  border: 1px solid black;
}
```

When you use `em`on the font-size rule, the font-size is set relative to the inherited font-size. So here, because the default document size is 16px, we get a font size of 32px and a padding of 64px.

```css
.box {
  font-size: 2em;
  padding: 2em;
  border: 1px solid black;
}
```

Defining font-size using `em`can cause problems when you have nested elements which all define font-size using `em`. This is because the inherited font-size is calculated from parent elements font-size value and passed down to child elements.

```css
div {
  font-size: 0.8em;
}
```

If you would now nest `div`elements they would have ever decreasing font-sizes.

This is where we can you another relative unit to fix things. The `rem`unit.

`rem`is a close relative to `em`because it also defines size relative to a font-size. `rem`however always uses to font-size of the root element on the document. By default the root font-size is 16px, but this depends on the default font-size the user has set on their browser.

```css
div {
  font-size: 0.8rem;
}
```

Now all our `div` elements would have the same font-size.

Using `em` and `rem`to define your lengths and font-sizes can be quite useful when you need to make responsive websites and make sure all the spacings and layouts work well on different screen sizes.

In general you should be using `em` to define things like height, padding, margin and using `rem`to define font-sizes. This allows easily changing the scaling and sizes of elements by changing the font-size and making sure the users browser settings are respected.

In addition to font-size based relative units CSS also has relative units which work in relation to the viewport size. These are `vh` (% of viewport height), `vw`(% of viewport width), `vmin`(smaller viewport dimension) and `vmax` (bigger viewport dimension).

`vh`and `vw` allow you to define a value as a percentage of either the viewport height or viewport width. Viewport here means the browsers area excluding addressbar, toolbars and frame. So you can for example define a hero element which fills always exactly 40% of the height of the viewport.

```css
.hero {
  height: 40vh;
}
```

With `vmin`and `vmax`you can use can define a value relative to which ever is the smaller or bigger viewport dimension. You could for example make sure a element will always fit in the viewport by defining it's size as relative to `vmin`.

```css
.box {
  height: 50vmin;
  width: 50vmin;
}
```

This will define a square box which size will always be half of the smaller viewport dimension.

There are some other relative units like `lh` (line height of element), `ex` (x-height of element), `ch`(width of 0 of the elements font) but we are not covering them here. For a more comprehensive article on CSS units you can checkout [MDN](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units)

