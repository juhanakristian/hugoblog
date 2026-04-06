---
title: 'Mocking GraphQL APIs with Mock Service Worker'
draft: false
date: '2021-02-14T09:33:01+00:00'
author: juhana.jauhiainen
tags: ["mocking", "testing", "graphql"]
description: "Mock Service Worker (MSW) is a library for mocking, or faking, a backend API. This is extremely useful when you are developing new features into your application, or when you are running tests."
---

Mock Service Worker (MSW) is a library for mocking, or faking, a backend API. This is extremely useful when you are developing new features into your application, or when you are running tests. 

In this article, I will guide you through setting up MSW for mocking a GraphQL API and show a few different kinds of ways you can mock queries and mutations. GitHub GraphQL API is used in the examples.

The example project was created using Create React App and Apollo Client. I won't be going through the UI or components in the example project but you can check the whole project in [GitHub](https://github.com/juhanakristian/react-graphql-msw-example) or [CodeSandbox](https://codesandbox.io/s/github/juhanakristian/react-graphql-msw-example).

---

## Setup MSW

MSW works by creating a [Service Worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) in the browser, catching the mocked HTTP queries and responding with the values you define. The service worker is defined in a special generated script that will need to be served from your applications `public` folder.

When running on Node.js (testing,) mocking is done by intercepting HTTP requests using `node-request-interceptor`, but in this article we will only be using browser mocking.

Before you start, install MSW using your favorite package manager. And create the service worker script to your applications `public` folder.
```shell
npm install msw --save-dev
npx msw init public
```

The last command will create a `mockServiceWorker.js` file into `public`. 

---

## Defining mocks

In our application directory, let's create a new directory named `mocks`
```shell
mkdir mocks
```

Within `mocks` we create a file called `handlers.js`. This file will hold our mock API definitions.

Inside `handlers.js` we need to import `graphql` from the `msw` library. This is a namespace that has the tools we need to mock GraphQL queries and mutations.

```js
import { graphql } from 'msw'
```

To mock an API that is not in the same domain as our app (`localhost`), we will use the `link` method.

```js
const github = graphql.link("https://api.github.com/graphql");
```

Now we can use the `github` object to define our query and mutation handlers. The query we will be mocking is the [repository](https://docs.github.com/en/graphql/reference/queries#repository) query. We define a operation called `RepositoryQuery` which takes two parameters: `repository` and `owner`. The query returns the `id`, `name`, `description`, and `stargazerCount` of the queried repository.

```js
const GET_REPOSITORY = gql`
  query RepositoryQuery($repository: String!, $owner: String!) {
    repository(name: $repository, owner: $owner) {
      id
      name
      description
      stargazerCount
    }
  }
`
```

Let's now define a mock handler for a `repository` query.

```js
export const handlers = [
  github.query("RepositoryQuery", (req, res, ctx) => {
    return res(
      ctx.data({
        repository: {
          id: "MDEwOlJlcG9zaXRvcnkzMzU0MTc5Mjc=",
          stargazerCount: 1,
          name: "next-graphql-msw-example",
          description:
            "A example of using MSW to mock GraphQL API in a NextJS app",
        },
      })
    );
  }),
];
```

This handler will simply wait for a query with the operation name `RepositoryQuery`, and respond with the JSON passed to `ctx.data` call. The handler is defined by calling `query` and passing the operation name and a handler function that will handle the query. The handler receives three parameters: `req`, `res` and `ctx`. 

`req` is an object containing information about the matched request.  
`res` is a function that can be used to return a response to the request.  
`ctx` is an object containing some helper functions.

To return a response, we can simply call `res` with an object and return its value.

Notice that even though the query is passing variables to the API, the handler is not using them, and it will always return the same data.


If we now perform the query in our application, we will get the response we defined in our mocks.

```js
 const { loading, error, data: queryData } = useQuery(GET_REPOSITORY, {
    variables: {
      owner: "juhanakristian",
      repository: "react-graphql-msw-example",
    },
  });

/* queryData
{
  repository: {
  id: "MDEwOlJlcG9zaXRvcnkzMzU0MTc5Mjc=",
  stargazerCount: 1,
  name: "react-graphql-msw-example",
  description: "A example of using MSW to mock GraphQL API in a React application",
}
*/
```

Nice! But what if we want to fetch data of another repository?

To achieve this, we need to access the variables in the query and return a different response.

```js
const { repository, owner } = req.variables;
if (repository === "msw" && owner === "mswjs") {
  return res(
    ctx.data({
      repository: {
        __typename: "Repository",
        id: "MDEwOlJlcG9zaXRvcnkxNTczOTc1ODM=",
        name: "msw",
        description:
          "Seamless REST/GraphQL API mocking library for browser and Node.",
        stargazerCount: 4926,
      },
    })
  );
}
```

`req.variables` contains the variables passed to the GraphQL query, and we can use those to decide what data will return.

---

## Enabling mocking

Next, we will need to set up the service worker to run when the app is started. To do this, add the next lines to `index.js`.

```js
if (process.env.REACT_APP_API_MOCKING === "enabled") {
  const { worker } = require("./mocks/browser");
  worker.start();
}
```

Now, when we start our app by running `REACT_APP_API_MOCKING=enabled npm start`, API mocking will be enabled, and our query will receive data from our handlers.

<div className="bg-yellow-200 text-black font-thin p-6 rounded-md flex gap-4 shadow-md">
<div>🙋 </div>
<div>To verify that mocking is working, check the developer console and if everything is working you should see <span className="text-purple">[MSW] Mocking enabled</span> printed in the console.</div>
</div>

## Mutations

Mutations are defined similarly to queries, but instead of the `query` method, we will be using the `mutation` method. The GitHub GraphQL schema has an `addStar` mutation that we can use to add a star to a repository. As a parameter, it takes an object of type `AddStarInput`, that contains the repository id in the `starrableId` attribute.

```js
const ADD_STAR = gql`
  mutation AddStarMutation($starrable: AddStarInput!) {
    addStar(input: $starrable) {
      clientMutationId
      starrable {
        id
        stargazerCount
        __typename
      }
    }
  }
`;
```

Let's now add the `AddStarMutation` operation to our handler and have it return data based on the `starrableId` variable passed in the mutation.

```js
github.mutation("AddStarMutation", (req, res, ctx) => {
  const {
    starrable: { starrableId },
  } = req.variables;
  if (starrableId === "MDEwOlJlcG9zaXRvcnkxNTczOTc1ODM=") {
    return res(
      ctx.data({
        addStar: {
          clientMutationId: null,
          starrable: {
            id: "MDEwOlJlcG9zaXRvcnkxNTczOTc1ODM=",
            stargazerCount: 4927, // Count increased by one!
            __typename: "Repository",
          },
        },
      })
    );
  }
  return res(
    ctx.data({
      addStar: {
        clientMutationId: null,
        starrable: {
          id: "MDEwOlJlcG9zaXRvcnkzMzgxNDQwNjM=",
          stargazerCount: 2, //Count increased by one!
          __typename: "Repository",
        },
      },
    })
  );
}),
```

Now, when we call the mutation, we will receive the updated `stargazerCount` from the handler, and our UI will also automatically update because Apollo will update its cache based on the returned `__typename` and `id`.


## Further reading

Mock Service Worker [docs](https://mswjs.io)  

If you have questions about Mock Service Worker, there's a channel for it in the [KCD Discord](https://kcd.im/discord)

Thanks for reading 🙏