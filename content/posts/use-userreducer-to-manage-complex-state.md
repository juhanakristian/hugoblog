---
title: "Use useReducer to manage complex state"
draft: false
date: "2020-10-31T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["react", "hooks", "reducer"]
description: "useReducer is a React Hook which helps you manage complex state with a simple API. The hook takes a reducer function and the initial state as parameters. You can also pass in an optional argument which can be a function for lazy initialization of the state."
---
`useReducer` is a React Hook which helps you manage complex state with a simple API. The hook takes a reducer function and the initial state as parameters. You can also pass in an optional argument which can be a function for lazy initialization of the state.

Here's the simplest example of using useReducer you can think of.
```js
import React from "react";

function reducer(value, newValue) {
  return newValue;
}

export default function App() {
  const [value, setValue] = React.useReducer(reducer, 0);
  return (
    <div className="App">
      <h1>{value}</h1>
      <button onClick={() => setValue(value + 1)}>Increment</button>
    </div>
  );
}
```

We are using `useReducer` for managing a single value, like when we use `useState`. But `useReducer` can do much more than that!

`useReducer` returns the current state and a function called **dispatcher**. The dispatcher is a special function that will call our reducer function with the value we give to it. In the previous example, the dispatcher calls the reducer with the new state value.
 
The reducer functions job is to calculate the new state from the old state. It does this using the value it's passed to by the dispatcher. Let's look at a more complex example.
```js
import React from "react";

function reducer(state, action) {
  switch (action) {
    case "increment":
      return state + 1;
    case "decrement":
      return state - 1;
    case "double":
      return state * 2;
    default:
      return state;
  }
}

export default function App() {
  const [state, dispatch] = React.useReducer(reducer, 0);
  return (
    <div className="App">
      <h1>{state}</h1>
      <button onClick={() => dispatch("increment")}>Increment</button>
      <button onClick={() => dispatch("decrement")}>Decrement</button>
      <button onClick={() => dispatch("double")}>Double</button>
    </div>
  );
}
```

Now our reducer supports three actions: increment, decrement, and double. Actions are operations performed to the state by the reducer. To perform these actions, we call dispatch and give it the name of the action we want to perform. The reducer function takes care of the state transform.

Our next example is going to add passing parameters to the reducer with our action.
```js
import React from "react";

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return state + 1;
    case "decrement":
      return state - 1;
    case "multiply":
      return state * action.mul;
    default:
      return state;
  }
}

export default function App() {
  const [state, dispatch] = React.useReducer(reducer, 0);
  return (
    <div className="App">
      <h1>{state}</h1>
      <button onClick={() => dispatch({ type: "increment" })}>Increment</button>
      <button onClick={() => dispatch({ type: "decrement" })}>Decrement</button>
      <button onClick={() => dispatch({ type: "multiply", mul: 5 })}>
        Multiply
      </button>
    </div>
  );
}

```

Now we pass the dispatcher an object with the action type and any parameters we want. This opens up a lot of possibilities! We can make actions that transform the state in many ways based on the parameters we pass.

The last example has a bit more complex state. 
```js
import React from "react";

function reducer(state, action) {
  if (action.type === "increment") {
    let updatedState = {};
    for (const key of state.selected) {
      updatedState[key] = state[key] + 1;
    }
    return {
      ...state,
      ...updatedState
    };
  } else if (action.type === "toggle") {
    if (state.selected.includes(action.fruit)) {
      return {
        ...state,
        selected: state.selected.filter((f) => f !== action.fruit)
      };
    } else {
      return {
        ...state,
        selected: [...state.selected, action.fruit]
      };
    }
  }

  return state;
}

const initialState = {
  selected: [],
  apple: 0,
  orange: 0,
  grapefruit: 0
};

export default function App() {
  const [state, dispatch] = React.useReducer(reducer, initialState);
  return (
    <div className="App">
      <div>
        <label htmlFor="apples">Apples</label>
        <input
          type="checkbox"
          id="apples"
          value={state.selected.includes("apple")}
          onChange={() => dispatch({ type: "toggle", fruit: "apple" })}
        />
      </div>
      <div>
        <label htmlFor="oranges">Oranges</label>
        <input
          type="checkbox"
          id="oranges"
          value={state.selected.includes("orange")}
          onChange={() => dispatch({ type: "toggle", fruit: "orange" })}
        />
      </div>
      <div>
        <label htmlFor="grapefruits">Grapefruits</label>
        <input
          type="checkbox"
          id="grapefruits"
          value={state.selected.includes("grapefruit")}
          onChange={() => dispatch({ type: "toggle", fruit: "grapefruit" })}
        />
      </div>
      <div>
        <button onClick={() => dispatch({ type: "increment" })}>Add</button>
      </div>
      <div>Apples: {state.apple}</div>
      <div>Oranges: {state.orange}</div>
      <div>Grapefruits: {state.grapefruit}</div>
    </div>
  );
}

```

Our state has a count of three fruits (apples, oranges, and grapefruits). The user interface has three checkboxes, one for each fruit. It also has a button for increasing the count of the selected fruits. For these interactions, we have two action types increment and toggle in our reducer. When a user selects fruits and clicks Add, the count of the selected fruits increases. When incrementing the counts, we use another state variable selected, to check which counts should be increased. 

Should you then use useState or useReducer? This depends on the structure of your state and whether the values of your state variables depend on other state variables. Sometimes moving from useState to useReducer simplifies the state management. Sometimes it results in a more complicated and hard to read code. Whether you stick with useState or move to useReducer should be decided on a case by case basis.  

If you want, you can check out the last example in [codesandbox.io](https://codesandbox.io/s/inspiring-shaw-2i4pm)
  
----
React documentation on useReducer  
[https://reactjs.org/docs/hooks-reference.html#usereducer]()

Kent C. Dodds  
Should I useState or useReducer?  
[https://kentcdodds.com/blog/should-i-usestate-or-usereducer]()
