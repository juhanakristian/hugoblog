---
title: "Basics of React server-side rendering with Express.js"
draft: false
date: "2022-02-05T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["react", "ssr", "expressjs"]
description: "If you want to develop SEO friendly and fast websites with React, you have two choices: server-side rendering (SSR) or static site generation (SSG). But how "
---

If you want to develop SEO friendly and fast websites with React, you have two choices: server-side rendering (SSR) or static site generation (SSG).

There are some awesome frameworks like [remix.run](https://remix.run/) , [next.js](https://nextjs.org/), [astro](https://astro.build/) or [11ty](https://www.11ty.dev/), which allow you to use one of (or both) techniques. So if you're building a production app, I recommend using one of them because server-side rendering is quite hard to get right.

**But** if you want to understand how it works and what is happening under the hood in these frameworks, you definately should try it out. This article will focus on how SSR works and we will also go through a simple example of using SSR.

### How React server-side rendering works?

Server-side rendering means **rendering the initial HTML on the server** instead of waiting for the JavaScript to be loaded in the browser and **then** rendering.

In client-side rendering, the browser makes a request for the `index.html` page, the server responds. the browser then reads this `.html` file and makes requests for any additional resources defined in it (CSS, JavaScript, favicon, images etc.). Only once the JavaScript is downloaded and can be executed, will there be anything rendered on the screen.

As we can see, the server doesn't really do anything here. That's why you can host a client-side rendered React app by just serving the static files using a web server like nginx.

![](/images/csr-diagram.png)

With server-side rendering, you need a server side application which handles the **initial** rendering of your React application. The server application will import your React applications root component and render it into a HTML document which is then returned to the client.

![](/images/ssr-diagram.png)

### Do I need to use server-side rendering in my React app?

If you're starting a new project and are serious about performance and SEO you should definately look into SSR. I'd recommend using one of the React frameworks tailored for SSR if they fit your needs.

For existing client-side rendered apps you should really weight the pros and cons. While SSR might provide some benefits (SEO, loading speed, social media previews), it will cost you some development time and will increase your server costs.

### How to implement server-side rendering

We're going to go through a simple, but limited, implementation of server-side rendering just to get you an idea on how it works.

You can use any Node.js or Deno framework for the server code but in this example, we're using Node.js, express and esbuild. The full source code of this example can be found [here](https://github.com/juhanakristian/react-ssr-example)

First let's look at the client side code.

Our main code in the client side is in `App.jsx`.

```jsx
import * as React from "react";

export default function App() {
  const [times, setTimes] = React.useState(0);
  return (
    <div>
      <h1>Hello {times}</h1>
      <button onClick={() => setTimes((times) => times + 1)}>ADD</button>
    </div>
  );
}
```

`App.jsx` contains a small React component with a counter which is increased when the user clicks a button. The only other file in the client side we need is an entrypoint.

```javascript
import * as React from "react";
import ReactDOM from "react-dom";
import App from "./App";

ReactDOM.hydrate(<App />, document.getElementById("root"));
```

`index.jsx` is the entrypoint for our client side code. Notice we're using [ReactDOM.hydrate](https://reactjs.org/docs/react-dom.html#hydrate) instead of [ReactDOM.render](https://reactjs.org/docs/react-dom.html#render). Instead of rendering the app (because it has already been rendered by the server) we are **hydrating** our app.

**Hydrating** refers to attaching all the event handlers to the correct elements of our server-side rendered DOM so our application will function correctly.

Next, let's take a look at the server code.

```jsx
import path from "path";
import fs from "fs";

import React from "react";
import ReactDOMServer from "react-dom/server";
import express from "express";

import App from "../src/App";

const PORT = process.env.PORT || 3000;
const app = express();

app.get("/", (req, res) => {
  fs.readFile(path.resolve("./public/index.html"), "utf8", (err, data) => {
    if (err) {
      console.error(err);
      return res.status(500).send("An error occurred");
    }

    return res.send(
      data.replace(
        '<div id="root"></div>',
        `<div id="root">${ReactDOMServer.renderToString(<App />)}</div>`
      )
    );
  });
});

app.use(
  express.static(path.resolve(__dirname, ".", "dist"), { maxAge: "30d" })
);

app.listen(PORT, () => {
  console.log(`Server is listening on port ${PORT}`);
});
```

On the server side, we use express to define a root endpoint which serves a `index.html` file. When a request is received, we render our React app root component `App` to a string using `ReactDOMServer.renderToString`. The rendered string is then injected into our `index.html` file so that we replace the div with the id `root` with our rendered content.

We also setup static file loading from `dist` folder so that our client side JavaScript code in `bundle.js` will be loaded once the browser reads `index.html`.

`index.html` contains basic HTML structure, a `<script>` tag for `bundle.js` and a `<div>` element which the React app will be rendered to.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>SSR App</title>
  </head>

  <body>
    <div id="root"></div>
    <script src="bundle.js"></script>
  </body>
</html>
```

Now, when a request is made to the root of our app, the express server renders our React app into a string and injects that into the HTML which is returned to the browser. The browser then loads our JavaScript file (`bundle.js`) which contains the `ReactDOM.hydrate` call. After `hydrate` is called, our application is fully interactive and works just like it before we moved to server-side rendering.

This setup is enough for a simple example, but falls down pretty quickly with a more complext app. For example, it has no support for routing, which means we would render the same HTML no matter which URL the user is loading. It's also missing setup for loading static files imported in React components.

### Summary

Server-side rendering is a useful technique you can use when you want to improve the load times and SEO of your React application. It is however, hard to implement well and might not be needed if your client-side rendered application is performing well and you don't have issues with SEO.

I strongly recommend you to try [remix](https://remix.run) or [next.js](https://next.org) if you want to build a server-side rendered app.

### Links

[React docs on ReactDOMServer](https://reactjs.org/docs/react-dom-server.html)  
[React docs on hydrate](https://reactjs.org/docs/react-dom.html#hydrate)  
[remix.run](https://remix.run)  
[next.js](https//nextjs.org)  
[Is SSR with React worth it? (Jakob Lind)](https://blog.jakoblind.no/is-ssr-worth-it/)
