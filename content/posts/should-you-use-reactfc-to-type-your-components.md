---
title: "Should you use React.FC to type your components?"
draft: false
date: "2021-12-07T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["react", "typescript", "webdev"]
description: "The consensus in the React community now seems to be that, you should avoid using React.FC. But why is that?"
---

**TLDR:** [No](https://github.com/facebook/create-react-app/pull/8177). [You](https://github.com/typescript-cheatsheets/react#function-components) [should](https://kentcdodds.com/blog/how-to-write-a-react-component-in-typescript) probably [not](https://twitter.com/nickemccurdy/status/1365384372908621833).

```ts
const Dialog: React.FC = ({children}) => {
	return (
		...
	)
}
```

The consensus in the React community now seems to be that, you should avoid using `React.FC` because it causes the component to implicitly take `children`. This means the component will accept children, even if they're not supposed to.

Let's take a look at the pros and cons of `React.FC`.

## Pros of React.FC

### Excplicit return type

In addition to the `children` prop, `React.FC` declares an explicit return type for the component. The return type is `ReactElement | null;` and we will get a type error if we try to return something else from the component.

```ts
const Header: React.FC = () => {
    return "test";
  )
}

/*
  Type '() => string' is not assignable to type 'FC<{}>'.
  Type 'string' is not assignable to type 'ReactElement<any, any> | null'.ts(2322)
*/

```

### Component static properties

Here's the full definition of `React.FunctionComponent` which `React.FC` is a shorthand for.

```ts
interface FunctionComponent<P = {}> {
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}
```

In addition to the function signature, `React.FC` provides type checking for some static properties. `propTypes` and `defaultProps` define the prop types based on the generic type `P`. These provide type validation for props you pass to the component and default prop values you can define with `Component.defaultProps = {}`.

`contextTypes` and `displayName` are less useful in most situations.
`contextTypes` is related to a [legacy Context](https://reactjs.org/docs/legacy-context.html).

`displayName` is used in [debug messages](https://reactjs.org/docs/react-component.html#displayname).

## Cons

### Implicit children

The biggest downside to using `React.FC` is that it causes the component to [implicitly take children](https://github.com/facebook/create-react-app/pull/8177). This means all components defined using `React.FC` will happily accept children, even when they're not supposed to.

```ts
const Icon: React.FC = <span className="icon">...</span>;

const OhNoh = <Icon>This should be a error</Icon>;
```

This won't be a runtime error, but TypeScript would catch this error if `React.FC` had not been used.

### Component as namespace

`React.FC` also doesn't accept generics (`GeneriComponent<T>`) and makes using the "component as namespace" pattern awkward to type.

```ts
<Dialog>
  <Dialog.Content></Dialog.Content>
</Dialog>;

const Dialog: React.FC<DialogProps> & { Content: React.FC<ContentProps> } = (
  props
) => {
  /*..*/
};
Dialog.Content = (props) => {
  /*...*/
};

//vs

const Dialog = (props: DialogProps) => {
  /*...*/
};
Dialog.Content = (props: ContentProps) => {
  /*..*/
};
```

## Alternatives

### Explicit props and return type

If you want to have the benefits of explicit return types, you can use `JSX.Element` to explicitly type the return value of your components. `children` should be typed with `React.ReactNode` since it accepts [all valid children values](https://github.com/typescript-cheatsheets/react#useful-react-prop-type-examples)

```ts
interface Props {
	children: React.ReactNode;
}

const Header = ({children}: Props): JSX.Element => {
    return "test";
  )
}

/*
  Type 'string' is not assignable to type 'Element'.ts(2322)
*/

```

### Explicit props but implicit return type

This is my preferred way of typing React components. I find the implicit return type is usually not a problem and haven't had any bugs where a component is returning something it's not supposed to.

```ts
interface Props {
	children: React.ReactNode;
}

function Header({children}: Props) {
	...
}

```

In my opinion, this is a pretty clear and clean way of typing React components. What this way loses in typesafe, it gains in readability. I also prefer using named functions because I think they are more readable than the array functions.
