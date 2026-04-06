---
title: 'Setup Spectron and Testing Library to effectively test your Electron.js application'
draft: false
date: '2021-03-14T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["testing-library", "electron", "javascript"]
description: "In this article, we will setup Spectron and use Testing Library and WebdriverIO to test an Electron.js application."
---

In this article, we will setup [Spectron](https://www.electronjs.org/spectron) and use [Testing Library](https://testing-library.com/) with [WebdriverIO](https://webdriver.io) to test an [Electron.js](https://electronjs.org) application.

Spectron is an open source framework for writing integration tests for Electron apps. It starts your application from the binary, so you can test it as a user would use it. Spectron is based on [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver) and [WebdriverIO](http://webdriver.io/).

Testing Library WebdriverIO is a library you can use to test web applications through [WebdriverIO](https://webdriver.io). It's part of the [Testing Library](https://testing-library.com) family of libraries.

With these tools we can write tests that run the application just like it's run by the user.

You should also consider using Testing Library to write unit/integration tests for your Electron.js application and its components. Since Electron.js applications use web technologies, this can be done in the usual way described in the Testing Library documentation.

However, I believe adding some tests that run the entire application from a binary can greatly increase your confidence that the application is working as intended.

## Example project

The example project used in this article was created using [Electron Forge](https://www.electronforge.io/) uses Electron 11, Spectron 12 and tests are run using Jest. I won't be covering every configuration step so if you want to know all the details, you can find the project at [github](https://github.com/juhanakristian/electron-spectron-example).


The example application has a header with the text *Hello from React!*, and a button with the text *Click me*. Once you click the button, another header with the text *Clicking happened* is added.

<div className="pt-4 pb-4">
  <img className="ml-auto mr-auto " src="/images/electron-spectron-example.png" />
</div>

## Setting up Spectron

First, we need to setup Spectron in our project

```shell
$ npm install --save-dev spectron
```

<div className="bg-yellow-200 text-black font-thin p-6 rounded-md flex gap-4 shadow-md mt-4">
<div>👉</div>
<div>
    Spectron is very nitpicky on the version of Electron.js you have in your project. A specific version of Spectron will only work with a specific version of Electron.js. For the matching version, check the <a href="https://github.com/electron-userland/spectron">Spectron GitHub</a>
</div>
</div>

Next, we'll create a basic test setup that will start our application before the test and close it after the test has run. To do that, we will initialize a Spectron `Application`. The initializer takes an options object as a parameter. The only attribute we will define in the options object is `path`. The `path` attribute is used to define the path to our application binary.

```js
import { Application } from "spectron";
import path from "path";

const app = new Application({
  path: path.join(
    process.cwd(), // This works assuming you run npm test from project root
    // The path to the binary depends on your platform and architecture
    "out/electron-spectron-example-darwin-x64/electron-spectron-example.app/Contents/MacOS/electron-spectron-example"
  ),
});

```

<div className="bg-yellow-200 text-black font-thin p-6 rounded-md flex gap-4 shadow-md mt-4 mb-4">
<div>👉</div>
<div>
If you don't have a binary to run yet, you need to run <span className="bg-yellow-400 p-1 rounded-md">npm run package</span> first to create a one. The <span className="bg-yellow-400 p-1 rounded-md">package</span> command is available if you've created your project using <a href="https://www.electronforge.io/">Electron Forge</a>
</div>
</div>


In the example, we have a setup in which the test is in `src/__test__/app.test.js`, so the binary is two directory levels up and in the `out` directory. We use `process.cwd()` to get the current working directory, which should be the project directory, and combine it with the path to the binary. The binary path will be different based on your platform and CPU architecture.

Now we can define a test setup that uses the `app` to start our application so we can test it.

```js
describe("App", () => {
  beforeEach(async () => {
    await app.start();
  });

  afterEach(async () => {
    if (app && app.isRunning()) await app.stop();
  });

});

```
We use the Spectron Applications `start()` and `stop()` methods for starting and stopping the application. They are asynchronous so we'll need to await them to be sure the app has started/stopped.

In this setup, the application is started before each tests so the previous test doesn't affect the execution of the next test. Your tests should always be independent and the order in which the tests are run shouldn't affect if they pass or not. 


## Adding a basic test

Let's now add a basic test to check that the application has started and the window is visible. This is what some would call a *smoke* test. It's purpose is to just check the application starts and no smoke comes out 😅  

We can do this by accessing `browserWindow` attribute on the `app` we created and by calling the `isVisible` method to check if the window is visible.

```js
test("should launch app", async () => {
  const isVisible = await app.browserWindow.isVisible();
  expect(isVisible).toBe(true);
});
```

When we run `npm test` we should see the application starting and closing immediately. The console should print the successful test result.

```shell
> electron-spectron-example@1.0.0 test
> jest .

 PASS  src/__test__/app.test.js (5.025 s)
  App
    ✓ should launch app (2336 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.005 s, estimated 8 s
Ran all test suites matching /./i.

```



## Troubleshooting

#### ❌ Application starting multiple times when running tests
If the application is started multiple times when you run the tests, this could be for a couple of reasons

**Mismatching Spectron and Electron versions**  
Check the versions from `package.json` and make sure they are compatible by checking Spectron [Github](https://github.com/electron-userland/spectron)

**Something is using the WebdriverIO port**  
By default port `9155` is used for WebdriverIO but if something is using it running the tests will wail. Change the port used for WebdriverIO when you initialize the Spectron `Application`
```js
const app = new Application({
  path: path.join(
    __dirname,
    "..",
    "..",
    "out",
    "electron-spectron-example-darwin-x64/electron-spectron-example.app/Contents/MacOS/electron-spectron-example"
  ),
  port: 9156,
});
```

## Setting up Testing Library WebdriverIO

Now we are ready to set up Testing Library WebdriverIO so we can use the awesome queries to test our application.

First, install the library
```shell
npm install --save-dev @testing-library/webdriverio
```

Next, we'll add another test to our existing `app.test.js` file. 

Import `setupBrowser` from `@testing-library/webdriverio`
```js
import { setupBrowser } from "@testing-library/webdriverio"
```

Now, let's add a test checking if the *Hello from React!* header is visible.

```js
test("should display heading", async () => {
  const { getByRole } = setupBrowser(app.client);

  expect(
    await getByRole("heading", { name: /hello from react!/i })
  ).toBeDefined();
});
```

In this test we first call `setupBrowser` and give it the `client` attribute of our Spectron `Application` instance. `client` is a WebdriverIO [browser object](https://webdriver.io/docs/browserobject). `setupBrowser` returns  dom-testing-library [queries](https://testing-library.com/docs/queries/about). In this case, we are using the `getByRole` query.

<div className="bg-yellow-200 text-black font-thin p-6 rounded-md flex gap-4 shadow-md mt-4 mb-4">
<div>👉</div>
<div>
The queries returned by setupBrowser are asynchronous so we need to use async/await in our test method.
</div>
</div>

Let's now add a test checking the button is working and the *Clicking happened* header appears like it's supposed to.

```js
test("should display heading when button is clicked", async () => {
  const { getByRole } = setupBrowser(app.client);

  const button = await getByRole("button", { name: /click me/i });
  button.click();

  expect(
      await getByRole("heading", { name: /clicking happened!/i })
  ).toBeDefined();
})
```

Here we use `button.click()` to emulate the user clicking the button. Normally with Testing Library, we would use `@testing-library/user-event` but with WebdriverIO it's not available so we need to use the [API provided by WebdriverIO](https://webdriver.io/docs/api/element/). 


Now we have three tests that give us confidence that our application is working like it's supposed to.

```shell
❯ npm t

> electron-spectron-example@1.0.0 test
> jest .

 PASS  src/__test__/app.test.js (10.109 s)
  App
    ✓ should launch app (2339 ms)
    ✓ should display heading (2639 ms)
    ✓ should add heading when button is clicked (2490 ms)

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        11.013 s
Ran all test suites matching /./i.
```

As you can see from the execution times of these tests (10s in total 😬), using Spectron might not be suitable for testing every aspect of your application. Instead you shoud make a small number of tests for the core functionality of your application.

Here is the full source code listing of `app.test.js`
```js
import { Application } from "spectron";
import path from "path";

import { setupBrowser } from "@testing-library/webdriverio";

const app = new Application({
  path: path.join(
    process.cwd(), // This works assuming you run npm test from project root
    // The path to the binary depends on your platform and architecture
    "out/electron-spectron-example-darwin-x64/electron-spectron-example.app/Contents/MacOS/electron-spectron-example"
  ),
});

describe("App", () => {
  beforeEach(async () => {
    await app.start();
  });

  afterEach(async () => {
    if (app && app.isRunning()) await app.stop();
  });

  test("should launch app", async () => {
    const isVisible = await app.browserWindow.isVisible();
    expect(isVisible).toBe(true);
  });

  test("should display heading", async () => {
    const { getByRole } = setupBrowser(app.client);

    expect(
      await getByRole("heading", { name: /hello from react!/i })
    ).toBeDefined();
  });

  test("should add heading when button is clicked", async () => {
    const { getByRole } = setupBrowser(app.client);

    const button = await getByRole("button", { name: /click me/i });
    button.click();

    expect(
      await getByRole("heading", { name: /clicking happened!/i })
    ).toBeDefined();
  });
});

```

## Further reading

- Spectron [docs](https://www.electronjs.org/spectron) and [GitHub](https://github.com/electron-userland/spectron)
- Testing Library WebdriverIO [docs](https://testing-library.com/docs/webdriverio-testing-library/intro)
- WebdriverIO [docs](https://webdriver.io/docs/api)
