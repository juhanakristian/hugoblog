---
title: 'How to make TypeScript understand Array.filter'
draft: false
date: '2021-10-04T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["typescript"]
description: "If you've ever used Array.filter to filter a list to certain type of items, you've probably been hit by TypeScript not realizing what type of items your list contains after filtering. To fix this, you can use user-defined type guards."
---

If you've ever used Array.filter to filter a list to certain type of items, you've probably been hit by TypeScript not realizing what type of items your list contains after filtering. To fix this, you can use user-defined type guards.

Let's say we have content items of two type: Post and Image.

```ts
interface Post {
	text: string;
    title: string;
    date: Date;
}

interface Image {
    url: string;
    alt: string;
    date: Date;
}
```

We are going to store items of both types in a array.

```ts

type Content = Post | Image;
const content: Content[] = [
    {
        title: "A post",
        text: "...",
        date: new Date()
    },
    {
        alt: "A image",
        url: "...",
        date: new Date()
    }
]
```

Now if we want to get a list of all the post titles, we can do it by first filtering filtering with `"title" in obj`. But even though we **know** this works and there's no way `title` is undefined, we still get a type error.

```ts
const postTitles = content.filter(c => "title" in c).map(c => c.title);
// Property 'title' does not exist on type 'Content'.
// Property 'title' does not exist on type 'Image'.(2339)
```

This is because TypeScript can't deduce that all the remaining items are of the type Post. We can fix this issue with a TypeScript feature called [user-defined type guards](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)

### What are user defined type guards?

Type guards allow narrowing a type with differenty ways. For example, you can use [typeof](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#typeof-type-guards) or [instanceof](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#instanceof-narrowing).

User-defined type guards are functions whose return type is a __type predicate__.

```ts
function isPost(content: Content): content is Post {
    return "title" in content;
}
```

Notice the return value of the function is `content is Post` this tells TypeScript the function will only return true if `content` is of type `Post`. The first part of a type predicate (here **content**) must be a name of parameter of the function.

Now we can use this type guard in our **Array.filter** and TypeScript won't yell at us anymore 🥳

```ts
const postTitles = content.filter(isPost).map(c => c.title);
```

If you want to play with the code in this article, checkout [this playground](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgAoHsDOZkG8BQAkJAB5gBcy2UoA5gNz7LPJjBgA2El1djLyACZxIlACIiIjAL758oSLEQoAkgFs4tFAQEBXKBx5gaIBkxZwOFKsb7nmw0cgmQZcsAE8ADigDC6cAhwZABeNCwcAB9kdU0pOQQA7GREwPBKfzSwAG0AXVDkbPs8YoE2Tm5kACIAQWQvCKqAGlKWUmsqgDpu5taHSUoQCAB3Z0kACgBKYukWgR0BCytKWuRgDS1exZZ9Q2ruzq3toQHkIdGXCCmZ-Fy5fBhdEAQ2ALXMDGxx1Mh05EzfmBJpQfkEcMBMOFkgtmFAIGB9CBquUuFU1kjQeA3Klkg1sAAVdhcSFhTFgTowYBWaDjCGfIGdDReb6hAB8KU6KIgk3oQA)

### Further reading
[Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)  
[Using type predicates](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)  