---
title: "Making API calls in React useEffect"
draft: false
date: "2020-05-16T09:33:01+00:00"
author: juhana.jauhiainen
tags: ["frontend", "react", "javascript"]
description: "useEffect is a hook added in React 16.8. It allows you to perform side effects in function components. This means you can update things outside of React based on your props and state. Fetching data when the component state changes, changing the page <title> or connecting to a WebSocket server are all examples of side effects that can be done with useEffect."
---

`useEffect` is a [hook](https://reactjs.org/docs/hooks-effect.html) added in React 16.8. It allows you to perform [side effects](<https://en.wikipedia.org/wiki/Side_effect_(computer_science)>) in function components. This means you can update things outside of React based on your `props`and `state`. Fetching data when the component state changes, changing the page `<title>` or connecting to a WebSocket server are all examples of side effects that can be done with `useEffect`.

As an example, we are going to build a component which fetches data from [Cat Facts API](https://alexwohlbruck.github.io/cat-facts/) and displays the received facts as a list. Finally we'll add buttons to select the animal we want facts about.

Let's start with a simple component which prints a message to console when the it is [mounted](https://stackoverflow.com/questions/31556450/what-is-mounting-in-react-js).

```jsx
function AnimalFactsList(props) {
  useEffect(() => {
    console.log("Hello from useEffect!")
  })
  return <div></div>
}
```

This seems to be working. "Hello from useEffect!" is printed to console when the component mounts. In fact React runs the function we supplied to `useEffect` every time it renders our component.

Next we will add state variable to hold the data we fetch, a API call for fetching it and display the results as a list of `<p>` elements.

```jsx
import React, { useState, useEffect } from "react"

function AnimalFactsList(props) {
  const [animalFacts, setAnimalFacts] = useState([])

  useEffect(() => {
    fetch(
      "https://cat-fact.herokuapp.com/facts/random?animal_type=cat&amount=5"
    )
      .then(response => response.json())
      .then(response => setAnimalFacts(response))
  })

  const facts = animalFacts.map(fact => <p key={fact._id}>{fact.text}</p>)

  return <div>{facts}</div>
}
```

If you run this you'll see something like this.

![API calls causing a render loop](/images/CatFactProblems.gif)

Uh oh.. Something is wrong here. The API is being called over and over and over again.

Remember, React is running our `useEffect` function every time it renders our component. The problem is.. we are changing the state of the component **in our side effect function**! And since React renders our component again when it's state has changed, we have created **render loop.**

From React the [documentation](https://reactjs.org/docs/hooks-effect.html#tip-optimizing-performance-by-skipping-effects) we find out we can skip running the effect by giving `useEffect`a second argument which defines the effects _dependencies_.

For now we want to run the effect only when the component mounts. This is what the documentation has to say.

> If you want to run an effect and clean it up only once (on mount and unmount), you can pass an empty array ([]) as a second argument. This tells React that your effect doesn’t depend on any values from props or state, so it never needs to re-run.

So let's add `[]` as the second parameter to `useEffect`.

```jsx
import React, { useState, useEffect } from "react"

function AnimalFactsList(props) {
  const [animalFacts, setAnimalFacts] = useState([])

  useEffect(() => {
    fetch(
      "https://cat-fact.herokuapp.com/facts/random?animal_type=${animalType}&amount=5"
    )
      .then(response => response.json())
      .then(response => setAnimalFacts(response))
  }, [])

  const facts = animalFacts.map(fact => <p key={fact._id}>{fact.text}</p>)

  return <div>{facts}</div>
}
```

Now the API is getting called only once when our component is mounded.

Next we'll add the ability to change the animal we're downloading facts about. We will add a few buttons, a new state variable and use the state variable in our API call.

```jsx
import React, { useState, useEffect } from "react"

function AnimalFactsList(props) {
  const [animalFacts, setAnimalFacts] = useState([])
  const [animalType, setAnimalType] = useState("cat")

  useEffect(() => {
    fetch(
      `https://cat-fact.herokuapp.com/facts/random?animal_type=${animalType}&amount=5`
    )
      .then(response => response.json())
      .then(response => {
        setAnimalFacts(response)
      })
  }, [animalType])

  const facts = animalFacts.map(fact => <p key={fact._id}>{fact.text}</p>)

  return (
    <div>
      <h1>Here's some facts about {animalType}s</h1>
      {facts}
      <button onClick={() => setAnimalType("cat")}>Cat</button>
      <button onClick={() => setAnimalType("dog")}>Dog</button>
    </div>
  )
}
```

Notice we've also added the new state variable `animalType`as a **dependency** to our effect. If we didn't do that the effect would be called only once when the component mounts but not after the `animalType`state variable changes.

This is a key concept of `useEffect`.

**You must add all the variables (props/state) the effect uses to the dependencies. If the dependencies are incorrect, the effect won't run when it's supposed to and the state variables inside the effect will have their initial values.**

Checkout the full code for this example at [codesandbox.io](https://codesandbox.io/s/eloquent-ellis-w0h12?file=/src/App.js)

To better understand `useEffect`and how functional components work in React I highly recommend reading Dan Abramovs excellent blog post [A Complete guide to useEffect](https://overreacted.io/a-complete-guide-to-useeffect/)
