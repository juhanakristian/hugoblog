---
title: "What are template literal types in TypeScript?"
draft: false
date: "2022-04-21T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["typescript", "web"]
description: "Template literal types are a powerful feature of TypeScript that you might not have heard of."
---

Before diving into template literal types, we first need to discuss [string literal types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types)

In TypeScript, string literal types are types that constrain the type of a variable to a string or a union of strings.

```ts
type Greeting = "hello world";

const text1: Greeting = "hello world"; // ✅  This is ok
const text2: Greeting = "hello"; // ❌ Type '"hello"' is not assignable to type '"hello world"'.
```

Constraining the value of a variable to a single string is not that useful but a union of strings is something you'll see often.

```ts
type GreetingType = "hi" | "hello" | "howdy";
function sendGreeting(greetingType: GreetingType, name: string) {}
```

**So what are template literal types then?** Well, just like in JavaScript, we can use [template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) to construct strings from variables, in TypeScript we can use a similar syntax to create new types. _A template literal type is a type created by combining types and strings using the template literal syntax._

```ts
type World = "world";

type Greeting = `hello ${World}`;
// type Greeting = "hello world"
```

Template literal types can be built using all simple types (or their unions) which can be converted to strings. However, using an object or an array type results in an error.

```ts
type Numbers = 1 | 2 | 3 | 4 | [1];

type NumberGreeting = `hello ${Numbers}`;
// Type 'Numbers' is not assignable to type 'string | number | bigint | boolean | null | undefined'.
// Type '[1]' is not assignable to type 'string | number | bigint | boolean | null | undefined'.(2322)
```

When we use a union with template literal types, we get a type that contains all combinations of the union and the string. And if we have multiple unions in our template literal type, it will contain **all the combinations** of those 🤯

Let's say we have a union of strings which represent translatable string ids and a union of strings which represent language ids. Now we want to create an object where the keys are combinations of string ids and language ids.

```ts
type MessageType = "error" | "notification" | "request";
type Lang = "en" | "fr" | "de" | "jp" | "cn";
```

We can use template literal types to create a union that includes all combinations of these.

```ts
type MessageTypeTranslations = `${MessageType}_${Lang}`;

/*
type MessageTypeTranslations = "error_en" | "error_fr" | "error_de" | "error_jp" | "error_cn" | "notification_en" | "notification_fr" | "notification_de" | "notification_jp" | "notification_cn" | "request_en" | "request_fr" | "request_de" | "request_jp" | "request_cn"
*/
```

We could now use this as a key type for an object which contains all the translations

```ts
type MessageTranslations = {
  [key in MessageTypeTranslations]: string;
};
```

### Type inference

Another cool feature of template literal types is we can also use them with type inference.

```ts
type Mouse = {
  position: [number, number];
  leftButtonState: boolean;
  rightButtonState: boolean;
};

type MouseEventName = `on${Capitalize<keyof Mouse>}Change`;
// Here R is a inferred type
type MouseEventKey<T> = T extends `on${infer R}Change`
  ? Uncapitalize<R>
  : never;
// MouseEventKey<"onPositionChange"> is equal to "position"
```

Here we are using template literal types to generate event names from the keys of the object type `Mouse`. Then we use type inference (in `MouseEventKey<T>`) to retrieve the original object key when given an event name.

We can now use these types to define a function type for an event listener.

```ts
type MouseEventFunc<T extends MouseEventName> = (
  value: Mouse[MouseEventKey<T>]
) => void;

// MouseEventFunc<"onPositionChanged"> is equal to (value: [number, number]) => void
```

And now we can define a function for adding event handlers and get type errors if the handler has an incorrect type.

```ts
function addListener<T extends MouseEventName>(
  event: T,
  callback: MouseEventFunc<T>
) {
  //...
}

// ✅  value is of type [number, number]
addListener("onPositionChange", (value) => {
  console.log(value[0], value[1]);
});

// ❌ Property '0' does not exist on type 'Boolean' (value is of type Boolean)
addListener("onLeftButtonStateChange", (value) => {
  console.log(value[0], value[1]);
});
```

### What can you do with template literal types?

Template literal string can be used to achieve some incredible stuff like [JSON parsing](https://twitter.com/buildsghost/status/1301976526603206657)or a working [SQL database engine](https://twitter.com/c_pick/status/1307433762914009090) 🤯 For more awesome template literal type stuff, check the [awesome-template-literal-type repo in GitHub](https://github.com/ghoullier/awesome-template-literal-types)

In my last article, I wrote on [how to implement useMediaQuery hook in React](https://juhanajauhiainen.com/posts/how-to-implement-usemediaquery-hook-in-react). The `useMediaQuery` hook takes a single parameter, a string with a [media query](https://developer.mozilla.org/en-US/docs/Web/CSS/@media), and returns whether the query is true. Since the parameter is just a string, it's easy to make a mistake with the query because we can't type check it. But now, with the knowledge of template literal types, we can!

This is what the typing for `useMediaQuery` looks like currently.

```ts
function useMediaQuery(query: string): boolean {
  //...
}
```

Type checking for all the possible [media features](https://developer.mozilla.org/en-US/docs/Web/CSS/@media#media_features) is outside of the scope of this article, but we can start by type checking for the most common one. By far, the most common use of media queries is checking if the width of the device is under a certain threshold with `max-width` or `min-width`.

First let's type the width queries

```ts
type CSSMaxWidth = `max-width: ${number}px`;
type CSSMinWidth = `min-width: ${number}px`;
type MediaWidthQuery = `(${CSSMaxWidth | CSSMinWidth})`;
// (max-width: 1024px) ✅
// (min-width: 768px) ✅
// (min-widht: 1024px) ❌ Type '"(max-widht: 1024px)"' is not assignable to type '`(max-width: ${number}px)` | `(min-width: ${number}px)`'.(2322)
```

Next, we combine the width queries with the possibility to define the [media type](https://developer.mozilla.org/en-US/docs/Web/CSS/@media#media_types) and we have a simple type for a media query.

```ts
type MediaType = "all" | "print" | "screen";
type MediaOperator = "and" | "not" | "only";

type MediaQuery =
  | `${MediaType} ${MediaOperator} ${MediaWidthQuery}`
  | MediaWidthQuery;
// (max-width: 768px) ✅
// screen and (max-width: 1024px) ✅
```

`MediaQuery` could already be used as the type for the `useMediaQuery` hook if we only want to allow a single query. If we want to check multiple media queries in a single string, we need to introduce some recursion 😎

```ts
type UseMediaQueryParam<Str, Orig> = Str extends MediaQuery
  ? Orig
  : Str extends `${MediaQuery},${infer Rest}`
  ? UseMediaQueryParam<Rest, Orig>
  : never;

function useMediaQuery<Param extends string>(
  query: UseMediaQueryParam<Param, Param>
): boolean {
  //...
}
```

Here, we first check if the parameter extends `MediaQuery`, which means it's just a single media query, and return the original parameter if so. If it's not a single media query, we check if it's a combination of a media query and a string separated with a comma. Then we pass it to `UseMediaQueryParam` recursively. If the string doesn't have a `MediaQuery` at the start, we return `never` because the string isn't a valid list of media queries.

![TypeScript flow](/images/template-literal-types-01.png)

### One last thing: string manipulation

Template literal types are a powerful feature of TypeScript and they become even more powerful with some of the built-in **string manipulation** types that come with TypeScript.

The types included are `Uppercase<StringType>`, `Lowercase<StringType>`, `Capitalize<StringType>`, and `Uncapitalize<StringType>`.

```ts
type MessageType = "error" | "notification" | "request";

type MessageId = `${Uppercase<MessageType>}_ID`;
// type MessageId = "ERROR_ID" | "NOTIFICATION_ID" | "REQUEST_ID"

type MessageName = `${Capitalize<MessageType>}`;
// type MessageName = "Error" | "Notification" | "Request"

type MessageIdLower = `${Lowercase<MessageId>}`;
// type MessageIdLower = "error_id" | "notification_id" | "request_id"

type MessageIdCapitalized = `${Capitalize<MessageIdLower>}`;
// type MessageIdCapitalized = "Error_id" | "Notification_id" | "Request_id"
```

String manipulation types make it easy to convert existing string literal types to new ones.

### Further reading

[TypeScript docs](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)  
[awesome-template-literal-type](https://github.com/ghoullier/awesome-template-literal-types)  
[I need to learn about TypeScript template literal types](https://dev.to/phenomnominal/i-need-to-learn-about-typescript-template-literal-types-51po)
