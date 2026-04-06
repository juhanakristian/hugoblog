---
title: 'Create a retro text effect with CSS'
draft: false
date: '2021-04-13T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["web", "css"]
description: "Ever wanted to create a cool 80's text with HTML and CSS? This article will show you how."
---

In this article, we're going to create an 80's style text effect with CSS. The effect uses multiple `text-shadow`, `background-clip`, `filter`, linear gradients and `-webkit-text-stroke` to achieve a cool metallic effect.

Before we dive into the actual CSS you probably want to see the end result.

<iframe src="https://codesandbox.io/embed/retro-3d-text-demo-v3q3h?fontsize=14&hidenavigation=1&theme=dark&view=preview"
     style={{width:"100%", height:500, border:0, borderRadius: 4, overflow:"hidden"}}
     title="retro-3d-text-demo"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

Let's start by defining our HTML markup that we will be using.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Retro effect with CSS</title>
  </head>
  <body>
    <div class="container">
      <h1 class="title">
        <span class="metal clip raise">Retro</span><br>
        <span class="metal clip raise">effect</span>
      </h1>
      <h2 class="subtitle">With CSS</h2>
    </div>
  </body>
</html>
```

The HTML markup has a container `div` element and two header elements. The `h1` element has two `span` elements that have the text *Retro* and *effect*. The text is split into to `span` elements so we can apply the effect to both of them individually. 

The `container`, `title` and `subtitle` classes define mostly positioning rules so we won't be covering them. You can find the full CSS code in the [GitHub repo](https://github.com/juhanakristian/retro-effect-with-css).

The `span` elements have two classes that we will use to define our effect. Let's create the metallic fill for the text by adding a `metal` CSS class.

Add the following code inside the `head` tags in the HTML.

```css
<style type="text/css">
  .metal {
    background: linear-gradient(
      hsl(239, 50%, 30%) 15%,
      hsl(239, 50%, 40%) 25%,
      hsl(200, 60%, 50%) 35%,
      hsl(100, 70%, 80%) 45%,
      hsl(60, 100%, 98%) 50%,
      hsl(240, 0%, 0%) 52%,
      hsl(240, 10%, 10%) 60%,
      hsl(240, 50%, 30%) 70%,
      hsl(220, 70%, 60%) 80%,
      hsl(212, 92%, 76%) 85%
    );
  }
</style>

```

This will define a background for our text consisting of a linear gradient with ten colors. The output will look like this.

![Metallic effect](/images/retro01.png)

The gradient goes from darker blue to a greenish almost white, then there's a hard transition to black and a gradient to ligh blue. This creates an effect of metallic reflection and shine.

But how do we use the gradient as fill for the text and not a background? This can be done using the `background-clip` rule. Let's add the following rules to our `.metal` class.

```css
.metal {
  /* ... */ 
  color: transparent;
  background-clip: text;
  -webkit-background-clip: text;
}
```

![Metallic effect using background clip](/images/retro02.png)

`background-clip: text` creates a clipping mask from the text and uses it as a mask when rendering the background. Only the area of the clipping mask is rendered. The text itself is also rendered, so we need to add `color: transparent` to prevent it from covering the background. For this to work on WebKit-based browsers we need to add `-webkit-background-clip: text`.

Next, as a polish 😜, we'll add a border to the text. If we would just add `border: ...` we would get a box-shaped border for the whole element, which is not what we want. 🤔

Luckily, there's a non-standard CSS rule `-webkit-text-stroke` which is [supported by all the major browsers except IE](https://caniuse.com/text-stroke). Firefox and Edge support the `webkit` prefix version and don't have their own prefix.


```css
.metal {
  /* ... */ 
  -webkit-text-stroke: 2px hsl(200, 65%, 80%);
}
```

![Effect with added text stroke](/images/retro03.png)

Nice! The only thing we have left is to add the 3D effect. We can achieve the effect by using a neat trick of stacking multiple shadows with increasing offset and changing color at every level of the stack. CSS has the `text-shadow` rule which we could use but unfortunately, it doesn't work well with `background-clip` because the shadow is drawn over the background 😔 

But the [CSS Filter effects](https://css-tricks.com/almanac/properties/f/filter/) also have a `drop-shadow` filter which we can use.

Let's add a new class `raise` to our CSS with the `filter` rule, that contains 5 `drop-shadow` filters for the 3D effect and one extra for an actual shadow.

```css
.raise {
  filter: drop-shadow(1px 3px 0px hsl(242, 54%, 10%))
    drop-shadow(1px 3px 0px hsl(242, 54%, 15%))
    drop-shadow(1px 3px 0px hsl(242, 54%, 20%))
    drop-shadow(1px 3px 0px hsl(242, 54%, 25%))
    drop-shadow(1px 3px 0px hsl(242, 54%, 30%))
    drop-shadow(0px 0px 10px rgba(16, 16, 16, 0.4));
}
```

![Finished effect](/images/retro04.png)

We're using the same color with increasing luminosity to create a gradient effect.  Because of the way the filter effects work, the offsets all have the same value. When we add multiple `drop-shadow` filters, each subsequent shadow uses the result of the previous one as the shadow outline. So we keep the offset at `3px` it will increase by `3px` on each step.

The `With CSS` text in the [CodeSandbox demo](https://codesandbox.io/s/retro-3d-text-demo-v3q3h?file=/index.html:1557-1894) uses two `text-shadows` to achieve its blue and yellow shadows.
I've left out the font and positioning CSS rules for simplicity.

```css
.subtitle {
  /* ... */
  transform: rotateZ(-10deg);
  color: hsl(318, 100%, 51%); /* pink */ 
  text-shadow: -2px -3px hsl(53, 100%, 81%) /* yellow */, 5px 5px hsl(240, 90%, 61%); /* blue */
}
```

### Learn more
[MDN on background-clip](https://developer.mozilla.org/en-US/docs/Web/CSS/background-clip)  
[MDN on text-shadow](https://developer.mozilla.org/en-US/docs/Web/CSS/text-shadow)  
[CSS-Tricks article on filter effects](https://css-tricks.com/almanac/properties/f/filter/)  
[CSS-Tricks article on -webkit-text-stroke](https://css-tricks.com/adding-stroke-to-web-text/)