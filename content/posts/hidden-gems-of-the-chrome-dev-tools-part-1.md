---
title: 'Hidden gems of the Chrome DevTools, Part 1: The Console API'
draft: false
date: '2021-04-03T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["web", "chrome", "tools", "debug"]
description: "Debugging, or finding the reason why your code doesn't work, is one of the most important skills a software developer needs. If you can debug effectively, you'll catch problems faster and even gain a better understanding on how things work under the hood."
---

Debugging, or finding the reason why your code doesn't work, is one of the most important skills a software developer needs. If you can debug effectively, you'll catch problems faster and even gain a better understanding of how things work under the hood. 

In frontend development, we have a variety of tools available to debug our code. We can use a [debugger](https://code.visualstudio.com/Docs/editor/debugging) to step through our code, we can log values to the browser console and we can use the [DevTools](https://developer.chrome.com/docs/devtools/) of our browser. 

This article series focuses on the Chrome DevTools and will go through some of the more unknown or experimental features. These might be features you'll add to your toolbox and use daily or something you might want to check occasionally when improving accessibility or performance.

In the first article of the series, I'll cover some less known methods of the Console API

## The Console API

As JavaScript developers, we are familiar with the  `log`, `warn`, and `error` methods of the [Console API](https://developer.chrome.com/docs/devtools/console/api/). But the Console API has many more methods that can be used when debugging. 

### trace()

`trace` can be used to print the current [stack trace](https://en.wikipedia.org/wiki/Stack_trace). You've probably seen a stack trace when an error has occurred in your application. Using `trace` you can print the current stack trace even if no error has occurred.

An example situation where you might use `trace` could be when you're unsure which place in your code is calling a method. 

```js
function someFunc() {
    console.trace();
    return "Hello!";
}

function otherFunc() {
    someFunc();
}

setTimeout(someFunc, 0);
otherFunc();

// Trace
//     at someFunc (/home/runner/BlandWelllitComments/index.js:2:13)
//     at otherFunc (/home/runner/BlandWelllitComments/index.js:6:5)
//     at /home/runner/BlandWelllitComments/index.js:9:1
//     at Script.runInContext (vm.js:130:18)
//     at Object.<anonymous> (/run_dir/interp.js:209:20)
//     at Module._compile (internal/modules/cjs/loader.js:999:30)
//     at Object.Module._extensions..js (internal/modules/cjs/loader.js:1027:10)
//     at Module.load (internal/modules/cjs/loader.js:863:32)
//     at Function.Module._load (internal/modules/cjs/loader.js:708:14)
//     at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:60:12)
// Trace
//     at Timeout.someFunc [as _onTimeout] (/home/runner/BlandWelllitComments/index.js:2:13)
//     at listOnTimeout (internal/timers.js:554:17)
//     at processTimers (internal/timers.js:497:7)


```

The actual trace you'll get depends on what kind of environment you run the code in. The trace in the example is actually from [repl.it](https://repl.it). The console API works mostly the same in Node.js and the browser. 

### assert([expression, errorMsg])

`assert` can be used to print an error message to the console if something unexpected happens. This is useful for example if you're writing a library. `assert` takes as parameters an expression and an object. If the expression evaluates as `false` - an error is thrown. The object is printed to the console along with the error.

```js
function doSomething(obj) {
    console.assert(obj.someProperty, "someProperty needs to be set!")
}

doSomething({});

// Assertion failed: someProperty needs to be set!
```

<div className="bg-yellow-200 mt-4 text-black font-thin p-6 rounded-md flex gap-4 shadow-md">
<div>❗️</div>
<div><span className="bg-yellow-400 p-1 rounded-md">console.assert</span> only prints the error message to the console. It does not do any error handling for you!</div>
</div>


### table([data])

`table` is a method that prints data as a table which is easier to read than just printing objects. This could be useful when you have lots of data and want to debug it.

```js
const data = [
    {
        city: "Tokyo",
        country: "Japan",
        population: 37_977_000
    },
    {
        city: "Jakarta",
        country: "Indonesia",
        population: 34_540_000
    },
    {
        city: "Delhi",
        country: "India",
        population: 29_617_000
    }
]

console.table(data)

// ┌─────────┬───────────┬─────────────┬────────────┐
// │ (index) │   city    │   country   │ population │
// ├─────────┼───────────┼─────────────┼────────────┤
// │    0    │  'Tokyo'  │   'Japan'   │  37977000  │
// │    1    │ 'Jakarta' │ 'Indonesia' │  34540000  │
// │    2    │  'Delhi'  │   'India'   │  29617000  │
// └─────────┴───────────┴─────────────┴────────────┘


```

You can also supply an array of fields to `table`, and only those fields will be printed.

```js
console.table(data, ["city", "population"])

// ┌─────────┬───────────┬────────────┐
// │ (index) │   city    │ population │
// ├─────────┼───────────┼────────────┤
// │    0    │  'Tokyo'  │  37977000  │
// │    1    │ 'Jakarta' │  34540000  │
// │    2    │  'Delhi'  │  29617000  │
// └─────────┴───────────┴────────────┘
```

### count([label]) and countReset([label])

`count` prints the number of times the method has been called on the same line with the same label. This can be useful when you want to find out how many times something is occurring.

```js
for (let i = 0; i < 100; i++) {
    const value = Math.random() * 100;

    if (value > 10)
        console.count("Value is over 10!", value);
}

// ...
// Value is over 10!: 84
// Value is over 10!: 85
// Value is over 10!: 86
// Value is over 10!: 87
// Value is over 10!: 88
// Value is over 10!: 89
// Value is over 10!: 90

```

If you want to reset the counter at some point, you can use `countReset`. You must supply it with the label if you used one with the `count` call.

### time([label]) and timeEnd([label])

If you're trying to find out what is causing poor performance, your first stop is probably the Chrome DevTools performance tab. Sometimes, however, it's useful to measure the time it takes to run some code in your application. This is where `time` and `timeEnd` become useful.

```js
console.time("random");

for (let i = 0; i < 10000; i++)
  Math.random();

console.timeEnd("random");
//random: 3.029ms
```

The methods accept a label that makes it possible to have multiple timings going on at the same time. If no label is provided, the label `default` is used.

### group([title]), groupCollapsed([title]) and groupEnd

If you're logging a lot of things it might be useful to group the `console.log` calls so they will be easier to view. This can be done with `console.group`.
`group` takes the title of the group. The following `console.log` calls will be grouped under the title.

```js
console.group("group1")

console.log("Starting random numbers")
for (let i = 0; i < 10; i++)
  console.log(Math.random() * 100)

console.groupEnd()

// group1
//   Starting random numbers
//   87.92193095845431
//   81.99300123275765
//   84.99678268072954
//   2.290929000620534
//   74.45009215115104
//   36.5278113066769
//   9.250056218875692
//   7.181886970350249
//   29.206363066629937
//   20.1791813157987
```

The above example shows the console print from Node.js. On the browser, the group will be printed with a handle for opening/closing the group. By default all groups are open, but using the `groupCollapsed` method you can print a group that is closed by default.

![console log of console.group](/images/group.png)

### clear

Finally, with `clear` you can clear the console. If you're printing a lot of things, for example in a loop, you might want to have only the latest `log` calls visible.

```js
for (let i = 0; i < 100; i++) {
  console.clear();
  console.log(`Index ${i}`)
}
//Index 99
```

In the browser, `clear` works only if `Preserve log` is not enabled in the DevTools settings.

## Learn more

[Console API Reference](https://developer.chrome.com/docs/devtools/console/)
