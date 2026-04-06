---
title: 'How to type an object with exclusive-or properties in TypeScript'
draft: false
date: '2021-09-12T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["typescript"]
description: "How can we define a type for an object, which has some required attributes and some attributes, of which one and only one must be defined?"
---

How can we define a type for an object, which has some required attributes and some attributes, of which one and only one must be defined?

Let's say we want to define a type for a message that has a `timestamp` and either a `text` or an `id`.

```ts
// ✅ This should be valid
const messageWithText = {
    timestamp: "2021-08-22T19:58:53+00:00",
    text: "Hello!"
}

// ✅ This should be valid
const messageWithId = {
    timestamp: "2021-08-22T19:58:53+00:00",
    id: 123
}

// ❌ This should be a type error 
const messageWithBoth = {
    timestamp: "2021-08-22T19:58:53+00:00",
    id: 123,
    text: "Hello!"
}

// ❌ This should be a type error 
const messageWithoutEither = {
    timestamp: "2021-08-22T19:58:53+00:00",
}
```

## Union type

One way to approach this would be to use a [union type](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types)

```ts
type Message = {timestamp: string}
type TextMessage =  {text: string};
type IdMessage = {id: number};
type TextOrIdMessage = Message & (TextMessage | IdMessage);
```

Now, `timestamp` is a required attribute, and one of `id` and `text` attributes must also be present in an object of type `TextOrIdMessage`.

```ts
const msg1: TextOrIdMessage = {timestamp: "2021-08-22T19:58:53+00:00", text: "Hello!"}
const msg2: TextOrIdMessage = {timestamp: "2021-08-22T19:58:53+00:00", id: 1234}

// ❌ Type error!
const msg3: TextOrIdMessage = {timestamp: "2021-08-22T19:58:53+00:00"}
```

This does not however prevent us from creating an object with both `id` and `text` 😔

```ts
// This is still valid..
const msg4: TextOrIdMessage = {timestamp: "2021-08-22T19:58:53+00:00", id: 1234, text: "Hello!"}
```

## Using never to forbid a key from a type

What we can do to prevent this, is to use the [never type](https://www.typescriptlang.org/docs/handbook/basic-types.html#never)

```ts
type Message = {timestamp: string}
type TextMessage =  {text: string, id?: never};
type IdMessage = {id: number, text?: never};
type TextOrIdMessage = Message & (TextMessage | IdMessage);
```

We've added `id` with the type **never** to `TextMessage` and `text` as type **never** to `IdMessage`. This means `id` and `text` can never be in the same object just like we wanted!

```ts
// ❌ Type error!
const msg4: TextOrIdMessage = {timestamp: "2021-08-22T19:58:53+00:00", id: 1234, text: "Hello!"}
```

This is a great solution and most of the time it would probably be enough. But is there a way to make a more generic, reusable solution? 

## A reusable exclusive-or utility type

As it turns out, there is. This is a solution I came across in a Stack Overflow question [Why does A | B allow a combination of both, and how can I prevent it?](https://stackoverflow.com/questions/46370222/why-does-a-b-allow-a-combination-of-both-and-how-can-i-prevent-it). The solution was provided by [jcalz](https://stackoverflow.com/users/2887218/jcalz?tab=profile).

The solution is a bit more involved than the previous one, so we will go through it step by step. Here's the full code to create a utility type called `ExclusifyUnion`. 

```ts
type AllKeys<T> = T extends unknown ? keyof T : never;
type Id<T> = T extends infer U ? { [K in keyof U]: U[K] } : never;
type _ExclusifyUnion<T, K extends PropertyKey> =
    T extends unknown ? Id<T & Partial<Record<Exclude<K, keyof T>, never>>> : never;
type ExclusifyUnion<T> = _ExclusifyUnion<T, AllKeys<T>>;
```

`ExclusifyUnion<TextOrIdMessage>` will give us a union type in which `id` and `text` are defined as optional undefined types. 

```ts
type ExclusifiedMessage = ExclusifyUnion<TextOrIdMessage>;
/*{
    timestamp: string;
    text: string;
    id?: undefined
} |
{
    timestamp: string;
    id: number;
    text?: undefined
}*/
```

That's a lot to unpack 😅 Let's look at `AllKeys` and `Id` first and then go through step by step, how `ExclisifyUnion` works.

On the first line `type AllKeys<T> = T extends unknown ? keyof T : never;` we are creating a utility for getting all the keys from a type. Just using `keyof T` isn't enough because in our example with the type `TextOrIdMessage` this would result in only the key `timestamp`. This is because **keyof a union type only returns the keys which are shared by all the types in the union.**

But with the magic of [distributive conditional types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types) we can extract all the keys from a union. This works because when `T extends unknown ? keyof T : never` is given a union, it applies `keyof T` to each part of the union. `AllKeys<TextOrIdMessage>` gives us a union of all the keys `"timestamp" | "id" | "text"`.

```ts
type AllKeys<T> = T extends unknown ? keyof T : never;
type AllTheKeys = AllKeys<TextOrIdMessage>
// "timestamp" | "id" | "text"
```

Now, let's look at the **Id** utility type.

```ts
type Id<T> = T extends infer U ? { [K in keyof U]: U[K] } : never;
```

At first look, **Id** looks pretty baffling 🤔 It seems to just give you back exactly the type you give to it..

```ts
Id<number> // number
Id<number[]> // number[]
Id<{a: string}> // {a: string}
```

But when you give **Id** a more complex type like `TextOrIdMessage`, the result is a "compiled" version type. While `TextOrIdMessage` and `Id<TextOrIdMessage>` are functionally the same, the latter is easier to work with because your editor can show you a more readable version of the type. 

The purpose of **Id** is to eliminate intersections in order to make the type easier to read. You can read more about it at the end of [this](https://stackoverflow.com/a/49683575/11849122) Stack Overflow answer.

```ts
Id<TextOrIdMessage>
/*
{
    timestamp: string;
    text: string;
} | {
    timestamp: string;
    id: number;
}
*/
```

How does **Id** work then? Well, it uses a conditional type with [infer](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types) to extract all the keys and value types from the given type in order to eliminate intersections.

```ts
type SomeIntersectionType = {id: number, timestamp: string} & {timestamp: string, text: string}
type ReadableIntersectionType = Id<SomeIntersectionType> // {id: number, timestamp: string, text: string}
```

Now that we've looked at what **AllKeys** and **Id** do, let's go over what **ExclusifyUnion** does to `TextOrIdMessage`.

```ts
type _ExclusifyUnion<T, K extends PropertyKey> =
    T extends unknown ? Id<T & Partial<Record<Exclude<K, keyof T>, never>>> : never;
type ExclusifyUnion<T> = _ExclusifyUnion<T, AllKeys<T>>;
```

The key to understanding **ExclusifyUnion** is noticing that it uses a [distributive conditional type](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types) to apply `Id<T & Partial<Record<Exclude<K, keyof T>, never>>>` to each part of a union given as the generic type `T`. Inside the conditional, `T` refers to the part of the union it is being applied to.

Remember, `K` here is `AllKeys<TextOrIdmessage>` which resolves to `"timestamp" | "text" | "id"`.

Our type **TextOrIdMessage**, is a union type with two parts, `{timestamp: string, text: string}` and `{timestamp: string, id: number}`.

So when we apply the inside of `Id<...>` to each part of **TextOrIdMessage**, we will get 

```ts
{timestamp: string, text:string} & Partial<Record<Exclude<"timestamp" | "id" | "text", "timestamp" | "text">, never>>
{timestamp: string, id:number} & Partial<Record<Exclude<"timestamp" | "id" | "text", "timestamp" | "id">, never>>
```

After resolving [Exclude](https://www.typescriptlang.org/docs/handbook/utility-types.html#excludetype-excludedunion) we will have

```ts
{timestamp: string, text:string} & Partial<Record<"id", never>>
{timestamp: string, id:number} & Partial<Record<"text", never>>
```

Resolving [Record](https://www.typescriptlang.org/docs/handbook/utility-types.html#recordkeystype) gives us

```ts
{timestamp: string, text:string} & Partial<{id: never}>
{timestamp: string, id:number} & Partial<{text: never}>
```

And finally resolving [Partial](https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype) and the intersection operators

```ts
{timestamp: string, text:string, id?: undefined}
{timestamp: string, id:number, text?: undefined}
```

The final result will be the union of these types just like we wanted 🥳

```ts
type ExclusifiedMessage = ExclusifyUnion<TextOrIdMessage>;
/*{
    timestamp: string;
    text: string;
    id?: undefined
} |
{
    timestamp: string;
    id: number;
    text?: undefined
}*/
```


## Further reading
[Why does A | B allow a combination of both, and how can I prevent it? - stackoverflow.com](https://stackoverflow.com/questions/46370222/why-does-a-b-allow-a-combination-of-both-and-how-can-i-prevent-it/61350902#61350902)  
[Proposal: Add an "exclusive or" (^) operator - github.com](https://github.com/Microsoft/TypeScript/issues/14094)  
[all possible keys of a union type - stackoverflow.com](https://stackoverflow.com/a/49402091/11849122)  
