---
title: "Convert an array to a map in JavaScript"
draft: false
date: "2021-07-06T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["javascript", "computer science", "reduce"]
description: "Sometimes it's useful to convert a array to a map for convenience or performance reasons. But how can we achieve that so that the resulting code will be easy to understand?"
---

Sometimes it's useful to convert a array to a map for convenience or performance reasons. But how can we achieve that so that the resulting code will be easy to understand?

I'm using the term **map** here to mean a data structure where a value can be accessed using an unique key. In JavaScript, objects can be used as maps but there also exists a special **Map** [type](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) which has some advantages and disandvantages when compared to using objects. We won't be covering **Map** in this article.

Let's say we have a array of projects we want to group by completion month. We might need to do this if we frequently access the projects of a certain month and don't want to search them from the array every time, or maybe we are rendering the projects into month components using React/Vue/Svelte. 

There's a couple of ways we can achieve this. First we will look at how to do this using the `Array.reduce()` method and then how to simplify things using `for..of`.

Here is the data we are going to be using in all the examples. It's a simple array of objects that have the attributes **name** and **completed**.

```js
const data = [
  {
    name: "Project 1",
    completed: "01-2021"
  },
  {
    name: "Project 2",
    completed: "02-2021"
  },
  {
    name: "Project 3",
    completed: "02-2021"
  },
  {
    name: "Project 4",
    completed: "02-2021"
  }
];
```

### Using Array.reduce()

`Array.reduce` takes two parameters, a function which is called for each element in the array  and an initial value for the return value of the operation. 

The function given to `Array.reduce()` should have the following signature `(accumulator, currentValue, index, array) => {...}`. 

`accumulator` is a value which is carried over from the previous calls to the function, `currentValue` is the value in the array we are currently at, `index` is the index we are currently at, and `array` is the array reduce was called on. You can omit `index` and `array` if you have no use for them.

The basic idea of **reduce** is that on every call, we use the value from `currentValue` to shape `accumulator` how we want until we have looped through all the values in the array. The return value of the function is set as the new value of the `accumulator` for the next iteration.

After the function is called on the last element in the array, the value of `accumulator` is returned as the return value of **reduce**.

Here's how we could filter group our data using **reduce**

```js
const projectsByMonth = data.reduce((result, project) => {
  const existingProjects = result[project.completed] || [];
  return {
    ...result,
    [project.completed]: [...existingProjects, project]
  }
}, [])

/*
{
  '01-2021': [ { name: 'Project 1', completed: '01-2021' } ],
  '02-2021': [
    { name: 'Project 2', completed: '02-2021' },
    { name: 'Project 3', completed: '02-2021' },
    { name: 'Project 4', completed: '02-2021' }
  ]
}
*/
```

`Array.reduce` gets the job done just fine, but the code is not the easiest to understand. While this example is a simple one, I almost always struggle to wrap my head around complex code which uses **reduce**. Could we make this better?

### Using for..of

`for..of` is a way to loop over any [iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) in JavaScript. We can use `for..or` to solve the same problem by creating the object beforehand and looping through the data. 

```js
let projectsByMonth = {};
for (const project of data) {
  const existingProjects = projectsByMonth[project.completed] || [];
  projectsByMonth = {
    ...projectsByMonth,
    [project.completed]: [...existingProjects, project]
  }
}

/*
{
  '01-2021': [ { name: 'Project 1', completed: '01-2021' } ],
  '02-2021': [
    { name: 'Project 2', completed: '02-2021' },
    { name: 'Project 3', completed: '02-2021' },
    { name: 'Project 4', completed: '02-2021' }
  ]
}
*/
```

While the code for adding the projects to the object is the same as in the previous example, the resulting code is a bit easier to understand because we are using a using a simple loop.

If you're concerded with performance you can also replace the object/array spread operators with `Array.push`.

```js
let projectsByMonth = {};
for (const project of data) {
  const key = project.completed;
  if (!(key in projectsByMonth))
    projectsByMonth[key] = []

  projectsByMonth[key].push(project)
}
```

Whether you find `for..of` easier to understand, is mostly a matter of taste and familiarity with **reduce**. However, thinking about clarity of the code we write is an important part of software engineering. Clear, explicit code will help other programmers (or ourselves in a couple of months 😜) understand the code and the reasoning behind it. 

I'll leave you with this quote from Martin Fowlers excellent book [Refactoring](https://refactoring.com)

> “Any fool can write code that a computer can understand. Good programmers write code that humans can understand.” 
> <span className="font-bold text-xl">Martin Fowler</span>




## Further reading

[MDN on reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)  
[MDN on for..of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)