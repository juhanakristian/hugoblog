---
title: 'A Tailwind CSS plugin for adding gradient masks'
draft: false
date: '2020-11-09T00:00:00+00:00'
author: juhana.jauhiainen
tags: ["tailwind", "css", "open source"]
description: "Some time ago I added excerpts to by blog. A small sample of the full blog text to is now shown on the front page after the post heading, date and tags. To make the exceprts look a bit nicer, I wanted add a effect which makes the text fade into the background."
---
Some time ago I added excerpts to my blog. A small sample of the full blog text is now shown on the front page after the post heading, date, and tags. To make the excerpts look a bit nicer, I wanted to add an effect that makes the text fade into the background.

You can do this with the CSS mask-image property by setting its value to a linear-gradient.

```css
.some-class {
  mask-image: linear-gradient(to top, rgba(0, 0, 0, 1.0) 0%, transparent 100%);
}
```

I use Tailwind CSS to style my blog so I wanted to find a solution which doesn't require adding CSS classes or files to my project.

After I didn't find a solution I decided this would be a good time to learn how to make Tailwind CSS plugins.

Creating the plugin was straight forward. I started by reading Tailwind [documentation](https://tailwindcss.com/docs/plugins) on writing plugins. Then I opened Tailwind Playground and added the plugin skeleton from Tailwind documentation to the config.

```js
// tailwind.config.js 
const plugin = require('tailwindcss/plugin')

module.exports = {
  plugins: [
    plugin(function({ addUtilities, addComponents, e, prefix, config }) {
      // Add your custom styles here
    }),
  ]
}
```

I was only adding utilities, so I removed `addComponents`, `e`, `prefix`, and `config` from the parameters. In the function body, I added my code for generating the CSS. I wanted to do all combinations of directions (top, right, bottom, left) and the ability to select the linear gradients start percentage. The percentages would go from 0% to 100% with 10% intervals.

Adding utilities with a plugin works by creating a JavaScript object with all the utility class names as attributes and the matching CSS as the values of those attributes.

So for a utility class of `gradient-mask-tr-50%`, I wanted to generate the following object

```json
{
  "gradient-mask-tr-50%": {
      maskImage: `linear-gradient(to top right, rgba(0, 0, 0, 1.0) 50%, transparent 100%)`
  }
}
```

The plugin has the directions and gradient steps configured as arrays. The code then uses a `reduce` function to loop through the directions and steps to generate the classes and CSS rules. The returned object is then passed to `addUtilities`. Here's the full source code listing of the plugin.

```js
plugin(function ({ addUtilities }) {
  const directions = {
    t: "to top",
    tr: "to top right",
    r: "to right",
    br: "to bottom right",
    b: "to bottom",
    bl: "to bottom left",
    l: "to left",
    tl: "to top left",
  };

  const steps = [
    "0%",
    "10%",
    "20%",
    "30%",
    "40%",
    "50%",
    "60%",
    "70%",
    "80%",
    "90%",
  ];

  const utilities = Object.entries(directions).reduce(
    (result, [shorthand, direction]) => {
      const variants = steps.map((step) => {
        const className = `.gradient-mask-${shorthand}-${step}`;
        return {
          [className]: {
            maskImage: `linear-gradient(${direction}, rgba(0, 0, 0, 1.0) ${step}, transparent 100%)`,
          },
        };
      });

      const stepClasses = Object.assign(...variants);
      return {
        ...result,
        ...stepClasses,
      };
    },
    {}
  );

  addUtilities(utilities);
})
```

Feel free to try the plugin in this [Tailwind Playground](https://play.tailwindcss.com/LmXt96GIul).

The next step was to create an installable npm package of the plugin code. I created a new npm project with `npm init` and copied my plugin code from the playground to a new file called index.js. It has a module.exports definition which contains my plugin function.

I had some confusion on what the module should export, because the plugins I checked didn't use Tailwinds `plugin` function, but instead wrapped the plugin function into another anonymous function. In the end, I realized the correct way for me was to export the result of calling the `plugin` function with my plugin code as a parameter.

Other than that, I added a readme with description and instructions and published the package to npm using `npm publish`.

The full source code can be found from the plugins GitHub repository.

Developing a Tailwind plugin was a simple process, especially because I could test the plugin with the playground before starting a project locally.

If you have any feedback on the plugin, you can add an issue in [Github](https://github.com/juhanakristian/tailwind-gradient-mask-image) or send me a message in [Twitter](https://twitter.com/juhanakristian).