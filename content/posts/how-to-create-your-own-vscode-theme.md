---
title: 'Creating a VS Code theme'
draft: false
date: '2021-01-25T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["vscode"]
description: "Ever wanted to create a theme for VS Code? It's simpler than you'd think but the process is a bit laborious and requires you to painstakingly go through all the attributes you need to change.

If you want your theme to work, and look nice, on multiple programming languages, you'll need to add a lot of color definitions.
"
---


Ever wanted to create a theme for VS Code? It's simpler than you'd think but the process is a bit laborious and requires you to painstakingly go through all the attributes you need to change.

If you want your theme to work, and look nice, on multiple programming languages, you'll need to add a lot of color definitions.

I've just released my first theme called [Lupine](https://marketplace.visualstudio.com/items?itemName=juhanajauhiainen.lupine) and this post goes how I created it and what I learned on the way.

## Generating theme files

Theme files are generated using the `Visual Studio Code Extension generator` which is a tool by Microsoft for creating VS Code extensions and themes.

You can install it by running
```shell
npm install -g yo generator-code
```

To generate a theme template run
```shell
yo code
```

This will start an interactive prompt that will guide you through the process of initializing a theme template.

![Theme generator](/images/theme-generate.png)


After answering the questions, a directory with the name of your theme will be created. Now you can open the directory in VS Code and start editing your theme.

```shell
cd my-theme
code .
```

The generated theme project has everything set up for you to start. In the `theme` folder you'll find a JSON file with the name of your theme. This JSON contains the default color definitions which you can start to modify.

---

## Developing your theme

The easiest way to test your theme is to go into debug mode (press `F5`). A new VS Code window is opened, and this window will use your new theme.

When you edit the colors in your theme `themes/my-theme.json`, you will see the results immediately in the debug window. Occasionally I had some trouble with the debug window not updating but that's easily dealt with by clicking restart in the `Debug Toolbar`

You can also use Chrome Dev Tools inside VS Code 🤯  to find out the current colors of the UI components. Opening Dev Tools is done by pressing `Cmd+Option+I` (`Ctrl+Alt+I` on Windows). In the DevTools you can find out the current color of a UI element, and then search with the color from your theme file and replace it.

This is a laborious process but I didn't find a better way to go about it. 😞

---

## Theming the UI

![Lupine theme](/images/vscode-ui.png)

Here are some of the UI colors I defined in my own theme.

### Editor

The editor is where you edit your code 🤣

``` json
    "editor.background": "#1B1929",
    "editor.lineHighlightBackground": "#8076C222",
    "editorCursor.foreground": "#988de4",
```

For the editor I defined `background`, `lineHightlightBackground` and `editorCursor.foreground` values. These define the editor background color, highlight color for the line where the cursor is, and the color of the cursor.

![Editor](/images/linehighlight.png)

### Activity Bar

The activity bar is found on the left side of the window and has big icon buttons for changing between different VS Code views. 

```json
    "activityBar.background": "#13111d",
    "activityBar.inactiveForeground": "#323b50",
    "activityBar.foreground": "#5c6e92",
    "activityBarBadge.background": "#8076C2",
```

For the activity bar I defined `background`, `inactiveForeground`, `foreground` and `activityBarBadge.background` values. The foreground keys define the color of the icons. `activityBarBadge` is for defining the colors of the small badges on the icons.

![Activity bar](/images/activitybar.png)

### Side Bar

The side bar is the one with files list, debug tools, etc. depending on the selected view.

```json
    "sideBar.background": "#1B1929",
    "sideBarTitle.foreground": "#bbbbbb",
    "sideBarSectionHeader.background": "#242c3f",
```    

These define some basic colors for the side bar which are used always independent of the content. The content needs to be styled independently. For the file list, I defined the following values.

```json
    "list.activeSelectionBackground": "#2a273f",
    "list.activeSelectionForeground": "#8076C2",
    "list.inactiveSelectionBackground": "#2a273f",
    "list.inactiveSelectionForeground": "#8076C2",
    "list.highlightForeground": "#8076C2",
    "list.hoverBackground": "#1f2636",
```

These define the color values for the file list items in their different states. `active` here refers to whether the VS Code window is active or not. The same `list` color definitions also apply to other places in the UI where there are lists.

![Side bar](/images/sidebar.png)

### Button

There aren't many buttons in the VS Code interface but I decided to add style to them so they don't stick out.

```json
    "button.background": "#8076C2",
    "button.hoverBackground": "#8076C299",
```

These values set the background for buttons both in normal and hover state.

![Button](/images/button.png)

### Status bar

The status bar is the bottom bar in the VS Code window. Depending on installed extension and configuration, the status bar shows the information on the current git branch, file encoding, etc.

```json
    "statusBar.border": "#1B1929",
    "statusBar.background": "#13111d",
    "statusBar.debuggingBackground": "#bb0000",
```

For the status bar, I used the same colors as the activity bar. I wanted to have the status bar a distinctive color when debugging is on so you can't miss it.

### Title bar

For the title bar, I used the same colors as the activity bar and status bar. 

```json
    "titleBar.activeBackground": "#13111d",
    "titleBar.inactiveBackground": "#13111d",
    "titleBar.inactiveForeground": "#5c6e92",
```

Those were the basic styling values that will get you started.

---
## Syntax highlighting

Taking inspiration from one of my favorite themes Night Owl. I started tinkering with the syntax highlighting colors. These are defined by adding definitions in the `tokenColors` section of the theme JSON. `tokenColors` is an array of objects which define `scope`, `fontStyle` and `foreground` for the code which matches the `scope`.

```json
{
    "name": "Comment",
    "scope": ["comment", "punctuation.definition.comment"],
    "settings": {
        "fontStyle": "italic",
        "foreground": "#637777"
    }
}
```

The scopes can be discovered by using VS Codes' internal tool `Inspect Editor Tokens and Scopes`. You can activate it by pressing `Cmd+Shift+P` and searching for "inspect editor".

### Palette

To get started with the palette for syntax highlighting I copied some colors from my favorite themes like [Night Owl](https://github.com/sdras/night-owl-vscode-theme) or [Nako](https://github.com/kettanaito/vscode-nako-theme). 

After reaching a somewhat satisfying result for JavaScript and TypeScript code, I copied all the colors to Figma and started tweaking them to my liking. The process slow and tedious, jumping between the debug VS Code to inspect tokens and editing color values in another VSCode window.

I ended up with a palette of 11 colors and basic syntax highlighting for JavaScript and TypeScript in React projects.

![Lupin palette](/images/stormfather-palette.png)

The syntax highlighting is in no way complete and I will be improving when it over time when I notice something that I don't like.

![Lupine syntax highlighting](/images/stormfather-syntax.png)

If you use my theme and find any issues, I would be very grateful if you report them in the projects [GitHub Issues](https://github.com/juhanakristian/lupine/issues).

If you want to create your own theme I recommend these resources  
[Sarah Drasners article "Creating a VS Code Theme"](https://css-tricks.com/creating-a-vs-code-theme/)  
[VS Code documentation on Theme Colors](https://code.visualstudio.com/api/references/theme-color)
