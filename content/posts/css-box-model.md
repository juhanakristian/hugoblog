---
title: "Use box-sizing: border-box for simpler element sizing"
draft: false
date: "2021-01-04T09:33:01+00:00"
author: juhana.jauhiainen
tags: ["css", "web"]
description: "The CSS box model and it's behaviour with different box-sizing values might be difficult to visualize."
---

The CSS property `box-sizing` defines how the `width` and `height` properties behave on a element with border or padding. Normally, when `box-sizing` has its defualt value of `content-box`, width and height affect the size of the **content**. This means that if you have a element with `padding: 10px`, `border: 8px` and `width: 100px` then full width of the element will be `136px`. This also affects elements which have a relative `width` or `height`.

In this example we have a container element with `width: 200px`. The child elements `width` is defined as `100%` but it also has a padding of `5em`

<div className="bg-orange-400 p-5 mb-5 mt-5" style={{width: 200}}>
  <div className="bg-blue-400 border-gray-900 border-solid border-8 box-content" style={{padding: 10, width: "100%"}}>
    content
  </div>
</div>

```css
.parent {
  width: 200px;
}

.child {
  width: 100%;
  padding: 10px;
  border: 8px;
}
```

This results in overflowing the container element. Let's now set the child element the property `box-sizing: border-box`

<div className="bg-orange-400 p-5 mb-5 mt-5" style={{width: 200}}>
  <div className="bg-blue-400 p-5 border-gray-900 border-solid border-8 box-border" style={{width: "100%"}}>
    content
  </div>
</div>


```css
.parent {
  width: 200px;
}

.child {
  box-sizing: border-box;
  width: 100%;
  padding: 10px;
  border: 8px;
}
```

Now the width of the child includes the width from `padding` and `border` so the child doesn't overflow the parent element anymore.

This makes using width and height much simpler because you don't have to think about how padding and borders affect the size. Some even recommend you set `box-sizing: border-box` as defaulf for all elements.

```css
* . *{
  box-sizing: border-box;
}
```

Whether you go this far is up to you but it's important to understand how width and height are calculated and how `box-sizing` affects that.


<!-- <div className="bg-orange-700 p-5">
<div className="bg-orange-400 p-5">
  <span >border</span>
  <div className="bg-green-400 p-5 h-">
    <span >padding</span>
    <div className="bg-blue-400 p-5 border-gray-900 border-solid border-4">
    <span >content</span>
    </div>
  </div>
</div>
</div> -->