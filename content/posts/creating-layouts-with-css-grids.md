---
title: 'Creating layouts with CSS grids'
draft: false 
date: '2021-06-16T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["css", "grid", "layout"]
description: "Grids are a powerfull feature of CSS but they can be a little overwhelming at first. In this article, we'll go through some basic concepts of CSS grids and learn how we can use grids to build layouts."
---

**Grids are a powerfull feature of CSS but they can be a little overwhelming at first. There are rows, columns, areas, lines, and multiple syntaxes for defining layouts. How do these concepts fit together and how to use them effectively?**

In this article, we'll go through some basic concepts of CSS grids and learn how we can use grids to build layouts. My goal is to shed some light on how to define grid areas grid lines, which I found somewhat confusing when learning CSS grids.

## Basics and terminology

A CSS grid is a `display` type that sets the layout of its children according to the grid [specification.](https://www.w3.org/TR/css-grid-1/)
Grids are built of **columns**, **rows**, **areas**, **cells**, **lines**, **tracks**, and **gutters**. To understand how these relate to each other, let's go through some definitions. So, bear with me 🐻

**Columns** are vertical tracks and contain the space between two vertical **grid lines**

**Rows** then are horizontal tracks and contain the space between two horizontal **grid lines**

What are **tracks** and **grid lines** then? Well, **tracks** and **lines** are how we specify our grid. A **track** is a space between two parallel lines and **lines** are created when we define the tracks for our grid. Tracks can be defined using `grid-template-columns`, `grid-template-rows`, and `grid-template-areas` CSS rules.

The smallest element in a CSS grid is a **cell**. It's an area defined by grid lines in all directions with no lines running through it.

**Gutters** are spacing between tracks and are defined using `column-gap`, `row-gap`, and `gap` CSS rules.

<CssGrid />

The grid in the example above is defined with the following CSS declarations

```css
grid-template-columns: [first-col-start] 1fr [first-col-end second-col-start] 1fr [second-col-end third-col-start] 1fr [third-col-end];
grid-template-rows: [first-row-start] 1fr [first-row-end second-row-start] 1fr [second-row-end third-row-start] 1fr [third-row-end];
```

`grid-template-columns` property defines our grid columns. The syntax for `grid-template-colums` has [multiple ways to define the columns](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-columns#syntax). In it's simplest form, `grid-template-colums` takes a list of sizes to use as column widths. For example, `grid-template-colums: 1fr 1fr 1fr` would create a grid with 3 columns of equal width. `fr` is a special grid unit that means a [fraction](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Basic_Concepts_of_Grid_Layout#the_fr_unit).

In our example, we are using a `grid-template-columns` syntax which has [named grid lines](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Layout_using_Named_Grid_Lines#naming_lines_when_defining_a_grid) using the bracket `[line-name]` syntax. Notice that some of the lines we've defined have **two** names, such as `[first-col-end second-col-start]`. Giving two names will make it easier to define our grid areas later on because of a special feature of CSS grids called **implicit grid areas**.

### Implicit grid areas

While you can name your grid lines with any [name](https://drafts.csswg.org/css-values-4/#custom-idents), I've chosen to end them with `start` or `end`. This may seem like a minor detail, but it unlocks 🔓 a "hidden" feature. Grid area names will now be [created for us](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Layout_using_Named_Grid_Lines#implicit_grid_areas_from_named_lines). So using grid line names `first-col-start` and `first-col-end` on adjacent lines will create a grid area named `first-col` 🥳 We can now use `first-col` when we are defining grid areas with `grid-column` or `grid-area` properties.

## Side panel layout with named grid lines


Next, let's take a look at a layout example with a navigation header, content area, sider panel, and a footer. We will define our [grid lines](https://developer.mozilla.org/en-US/docs/Glossary/Grid_Lines) by name and using those names when we define the areas of the different layout components.

<SideNavExample/>

The thing to note is that the browser will automatically create [grid areas](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Grid_Template_Areas) for us when we define grid lines. Vice versa, if we define our grid using `grid-template-areas`, we will automatically get named grid lines generated for us.

To get started, here's the HTML structure we will be working with.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Side panel layout</title>
  </head>
  <body>
   <nav>nav</nav>
    <main>
      main
    </main>
    <aside>
      side
    </aside>
    <footer>
      footer
    </footer>
  </body>
</html>

```

We will set the `html` and `body` height to 100% with no `padding` or `margin` so the grid will have the full size of the page.

```css
html {
    height: 100%;
    padding: 0;
}
body {
    height: 100%;
    padding: 0;
    margin: 0;
}
```

Next, we'll add `display: grid` to the body and define our column **grid lines** using the `grid-template-columns` CSS rule. This declares two columns that are defined by four named grid lines: `side-start`, `side-end`, `main-start`, and `main-end`. The first column will be `200px` wide, while the other column has a width of `auto`. 

```css
body {
    /* ... */
    display: grid;
    grid-template-columns: [side-start] 200px [side-end main-start] auto [main-end];
    grid-template-rows: [nav-start] 50px [nav-end content-start] auto [content-end footer-start] 80px [footer-end];
}
```

When we use `grid-template-colums` and `grid-template-rows` with grid line names like this, named columns and rows will be automatically generated for us 🥳. In this case, the grid we will get the columns `start`, `side`, and `main` and the rows `nav`, `content`, and `footer`.

If we didn't care about the column and row names, we could just write `grid-template-columns: 200px auto` and `grid-template-rows: 50px auto 80px` to define the same layout.

Next we will define how our HTML elements will fit into the columns and rows. 

```css
nav {
    grid-row: nav;
    grid-column: side / main;
}
```

Because we have the generated row and column names, declaring the nav-element position and size can be done with `grid-row` and `grid-column` declarations. We are setting `grid-column` to `side / main` which tells the browser to extend the nav element from the start of the `side` column to the end of the `main` column.

```css
main {
    grid-area: content / main;
}
```

Defining the position and size of the main-element can be done using a single `grid-area` declaration. `grid-area` is a shorthand for setting `grid-row-start`, `grid-column-start`, `grid-row-end`, and `grid-column-end`. Because we are setting only two values (content and main) this is the equivalent of setting the row and column ending to the same values as start. 

```css
main {
    grid-row-start: content;
    grid-row-end: content;
    grid-column-start: main;
    grid-column-end: main;
}
```

You could also use the **grid lines** to define these values.

```css
main {
    grid-row-start: content-start;
    grid-row-end: content-end;
    grid-column-start: main-start;
    grid-column-end: main-end;
}
```

For the sidebar we will use `grid-area` and `grid-row-end` to define the area.

```css
aside {
    grid-area: content / side;
    grid-row-end: footer;
}
```

The `grid-area` declaration is similar to the one we used before. With the `grid-row-end` declaration, we make the sidebar extend to the bottom of the screen to the end of the **footer** row. This is cool, because it shows how powerful CSS grids can be 😎

Lastly, we define the footer CSS.

```css
footer {
    grid-row: footer;
    grid-column: main;
}
```

Here we're using the `grid-row` and `grid-column` declarations. This is the equivalent of writing `grid-area: footer / main`.

Now we have all the CSS we need for our layout 🥳 If you want to check out the full HTML/CSS code, you can check the out at [codesandbox.io](https://codesandbox.io/s/damp-smoke-7yfpy?file=/side-panel.html). There you'll also find two other layout examples made using CSS grids.

<iframe src="https://codesandbox.io/embed/damp-smoke-7yfpy?fontsize=14&hidenavigation=1&initialpath=side-panel.html&module=%2Fside-panel.html&theme=dark"
     style={{width: "100%", height: "500px", border:0, borderRadius: "4px", overflow:"hidden"}}
     title="damp-smoke-7yfpy"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## There's more to it

I hope this article has shed some light on creating layouts using CSS grids. There are, however many features we didn't cover. I've compiled here a few resources which you might find interesting if you want to dig deeper.

[MDN on CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout)  
[CSS-Tricks - A complete guide to grid](https://css-tricks.com/snippets/css/complete-guide-grid/)  
[Build Modern Layouts using CSS Grid - Hiroko Nishimura](https://egghead.io/courses/build-modern-layouts-with-css-grid-d3f5)  
[Build Complex Layouts with CSS Grid Layout - Rory Smith](https://egghead.io/courses/build-complex-layouts-with-css-grid-layout)  