---
title: 'How to loop over a JavaScript object'
draft: false
date: '2020-04-30T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["javascript"]
description: "There's a few ways to loop over a object in JavaScript. Let's look at a couple of them.

Let's say we have a object like this"
---
There's a few ways to loop over a object in JavaScript. Let's look at a couple of them.

Let's say we have a object like this

```javascript
const animals = {
    cat: "meow",
    dog: "woof",
    duck: "quack",
    pig: "oink",
    turkey: "gobble"
}
```

One way to loop over a object is to use the `Object.keys()` function to first retrieve the keys in a object and then loop over the returned array.

```javascript
const keys = Object.keys(animals);
for (let i = 0; i < keys.length: i++) {
  const animal = keys[i];
  const sound = animals[animal];
  console.log(`A ${animal} says ${sound}!`);
}
// A cat says meow!
// A dog says woof!
// A duck says quack!
// A pig says oink!
// A turkey says gobble!
```

You could also use Array function `forEach` for looping the array but in ES6 the `for..of` syntax was introduced for looping iterators (including arrays).

```javascript
const keys = Object.keys(animals);
for (const animal of keys) {
  const sound = animals[animal];
  console.log(`A ${animal} says ${sound}!`);
}
// A cat says meow!
// A dog says woof!
// A duck says quack!
// A pig says oink!
// A turkey says gobble!
```

`Object` also has as the function `Object.entries()` which returns a array of 2 element arrays in which the first element is a key in the object and the second element is the corresponding value. Using `Object.entries()` is convenient when you want to access both the key and value at the same time.

```javascript
for (const [animal, sound] in Object.entries(animals)) {
  console.log(`A ${animal} says ${sound}!`);
}
// A cat says meow!
// A dog says woof!
// A duck says quack!
// A pig says oink!
// A turkey says gobble!
```

The last way to loop over a JavaScript object is using the `for..in` syntax. It allows us to loop trough the object keys without extracting the keys first.

```javascript
for (const animal in animals) {
  const sound = animals[animal];
  console.log(`A ${animal} says ${sound}!`);
}
// A cat says meow!
// A dog says woof!
// A duck says quack!
// A pig says oink!
// A turkey says gobble!
```

Most of the time I prefer the `for..in` syntax becaus the `for (...)` sentence becomes clean and simple. To access the value you however have to extract it inside the loop using the square bracket syntax `object[attribute]`.
