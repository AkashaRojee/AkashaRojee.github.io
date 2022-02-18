---
layout: post
title: "Testing React Components with API Calls and Redux Store"
date: 2021-09-04
---

This guide shows how to mock API calls and use the Redux store when testing React components.

For tests to be effective, they have to be accurate: "the more your tests resembles the way your software is used, the more confidence they can give you" (Kent C. Dodds).

When your tests deal with dynamic behaviour and side effects, you want to mimic the real objects in the best way possible.

**The test scenario that we'll use in this guide:**

**_You have a React component that makes an API call and loads the returned data into a Redux store, and you want to test the component using React Testing Library (RTL)._**

While researching into the best way to do this, I came across service workers and network-level interception, and custom RTL renders. This guide lays down my findings and approach to the test scenario.

We'll have a deeper look into the test scenario, then briefly look at the different ways to mimic real objects in tests.

Before getting to the how-to, we'll go through the reasoning behind the best way to test thunks and asynchronous logic, to mock API requests, and to make the Redux store globally available throughout the test environment.

The how-to section will then walk you through how the test setup works and the steps for implementing it, along with a demo.

## Test scenario

You have a React component that makes an API call and loads the returned data into a Redux store, and you want to test the component using RTL.

### Assumptions

- Create React App (CRA) is used to set up the app.
- The API is a REST API.
- Jest is used as test runner.

### Constraints

During testing, we don't want to hit the API. We might exceed API limits. In the case of POST requests, we might even edit data in the API. So, we have to mock the API requests.

Moreover, fetching data from an API is asynchronous logic. In Redux, the standard is to place asynchronous logic inside thunks. So, API calls are made inside thunks.

### Questions

How do we test thunks? How do we properly mock asynchronous requests?

## Test doubles

First, let's have a brief look at test doubles. A test double is a way to mimic a real object for testing purposes. Some of the most common ones are:

- Stubs (you hardcode data and use that in your tests)
- Fakes (you use a simplified implementation of the object)
- Mocks (you preprogram an object that can respond dynamically)

Understanding the differences between them is essential to know when to use each in your tests.

Stubs and fakes do not allow you to test the real behaviour of your app. Stubs are only predefined data that are used as answers to calls during your tests. As for fakes, though they are working implementations, they have limited capabilities and are not the same as the actual implementation in your code.

On the other hand, mocks allow you to verify that the expected actions are performed. E.g. API requests/responses being sent/received properly.

## Testing thunks?

Thunks are an implementation detail (something which your users don't use/see/know about).

React recommends testing your components without relying on their implementation details. In other words, your test should focus on whether your components work the way they should for your users.

One of the guiding principles of RTL is actually that tests should be user-centric - tests should use components the way users would use the app. As such, implementation details are not to be tested in isolation.

This prevents your tests from breaking when you refactor your components (whereby implementation details are changed, but functionality remains the same), and prevents false positives (whereby your tests pass even though your components are not working as they should).

Because this testing approach makes you look at your app from the user's perspective, it also encourages you to apply accessibility best practices in your components.

So, if we do not test thunks, how do we test asynchronous logic, such as API requests, which are located inside of thunks?

## Testing asynchronous logic

You could hardcode a resolved promise with sample API data as value, and feed that as input to your test - this is an example of a stub. However, this would at most allow you to test that data is being processed properly, and not how your app actually handles asynchronous logic.

Ideally, when testing asynchronous logic located inside of thunks, we want the test to still follow thunk behaviour, i.e. the asynchronous logic inside the thunk must be able to interact with the Redux store's `dispatch` and `getState` methods. This allows you to test the real behaviour of your app.

Here's what Redux recommends:

- Cover the thunk by testing the component / group of components using it.

- Mock asynchronous requests at the network level.

## Mocking API requests

Mocking API requests at the network level makes it so that thunk logic does not have to change in the test.

The thunk will still try to make a real API request, which then gets intercepted by the mock implementation, after which the thunk proceeds normally.

**Note:** Redux recommends to use Redux Toolkit's `createAsyncThunk` to write thunks instead of writing them manually. This way, the thunk can handle promise-based action types of the API response (`pending`, `fulfilled`, `rejected`).

Since network-level API mocking involves intercepting a real API request to return a mock response, why not just mock the API request itself?

You don't expect your users to mock fetching data from an API. Going by the RTL's guiding principles, your tests shouldn't mock fetching either.

At best, you could mock fetching by reimplementing the API as a function and setting this as the mock implementation of the window's fetch, using spies - this is an example of a fake.

However, if the API's implementation changes (e.g properties and headers of the fetch response), you'll have to change your mock function. Moreover, if there's an error with the way you call your fetch in the test, the test could still pass, but it'd actually be a broken test.

The better solution is therefore to test API calls with implementation details out of the picture.

<!-- ...That way, you can still refactor your API calls, and your tests would... -->

## API mocking libraries

Enter API mocking libraries that operate at the network level.

This way, your test will make a real API request, and the mocking library will intercept it and return the mock response that you specified for the given API endpoint. You won't need to mock the implementation of the API itself, nor the API request.

Some of the tools that Redux recommends are Mock Service Worker, MirageJS, Jest Fetch Mock, and Fetch Mock. The one I use is Mock Service Worker (MSW).

MSW is an API mocking library that uses the Service Worker API (standard available on all modern browsers) to intercept actual requests. Here, the service worker acts like an intermediary server between your app and the network, and intercepts network requests from your app.

Thanks to service workers, MSW allows you to mock at the highest level of the network communication chain, and is the closest thing to mocking a server without creating one. Your app will behave the same as in production, and its code doesn't need to change to accomodate mocking. Moreover, the requests can still be observed in your browser's developer tools.

Additional benefits to this type of mocking are that it can also be used in other scenarios:

- Development - if the actual API is not ready yet, you could mock the API server for your app to use meanwhile
- Debugging - if a specific API scenario is causing an issue in your app, you could mock it to debug it
- Experimentation - you could try different API architectures in your app without having to rewrite the API

## How MSW works

1. MSW registers a service worker that listens to your app's network requests.

2. During your test, when the thunk makes the API request, the service worker intercepts the request and sends it to MSW.

3. MSW returns the mock response (specified beforehand in MSW configuration) to the service worker.

4. The service worker sends that response to your test.

### MSW How-To

Step 1: Installation

Install MSW:

```
npm install msw
```

If you use CRA, RTL should already be present. Otherwise, install it:

```
npm install @testing-library/react
```

If you use Jest, it is also recommended to use Jest DOM to access additional matchers that you can extend Jest with for RTL.

If you use CRA, Jest DOM should already be present. Otherwise, install it:

```
npm install @testing-library/jest-dom
```

Step 2: Configure MSW

You need to define which requests should be mocked, and what mock reponses they should return. To do this, you'll use **request handlers**.

You also need to have a **server** to handle those requests and responses.

You'll start by **defining mocks for the handlers and the server**.

In `src`, create a `mocks` folder to store your mock definitions.

2.1. Configure handlers

In `/mocks`, create a `handlers.js` file.

Since the API is a REST API, you'll use MSW's `rest` namespace to handle the requests.

Import `rest` from the `msw` package.

```
//handlers.js

import { rest } from 'msw';
```

To handle a **REST** API request, you need to specify
- its **method** (`get`, `post`, `put`, `patch` or `delete`)
- its **path** (the API URL endpoint)
- its **response resolver** (a function that returns the mocked response)

First, create an array to store the handlers.
- For each request that you want to mock, create a handler and specify the path.
- Each handler is in the form of `rest[method](path, responseResolverFunction)`.

```
//handlers.js

export const handlers = [
  
  rest.get('https://www.api-url.com/endpoint', responseResolverFunction),

  rest.post('https://www.api-url.com/endpoint', responseResolverFunction),

  // etc
];
```

As for the response resolver, the function has 3 arguments:
- **req** (request): information about the captured request
- **res** (response): function that creates the mocked response
- **ctx** (context): functions that set properties (e.g. status, headers, body, etc.) of the mocked reponse

Provide response resolvers to the request handlers.

```
//handlers.js

export const handlers = [
  
  rest.get('https://www.api-url.com/endpoint', (res, res, ctx) => {
    
    return res(
      ctx.json(
        //insert mocked data here
      ),
    );

  },

  rest.post('https://www.api-url.com/endpoint', responseResolverFunction),

  // etc
];
```
*For a full example of response resolvers, see <a href="https://mswjs.io/docs/getting-started/mocks/rest-api#response-resolver" target="_blank">this guide</a>.*

Now that the request handlers are configured, you need to integrate the mocking by configuring a server.

2.2. Configure server

In `/mocks`, create a `server.js` file.

Jest is a non-browser, NodeJS environment, and service workers cannot run in such environments. So, you need to set up a request interception layer for a NodeJS environment.

Import `setupServer` from `msw/node`, and the handlers from `handlers.js`. Then, create a server instance and pass in the handlers.

```
// server.js

import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

**Note:** the server is not an actual server but a mock server. The mock server will intercept requests specified in the handlers, and will handle the requests as if it were a real server.

Step 3: Configure API mocking

Now that MSW is configured, you have to configure API mocking for your tests.

**During your tests, you want to focus only on testing.** You don't want to have to reference any mocking. Because of this, it's recommended to configure API mocking during **test setup**.

CRA provides you with a `setupTests.js` module. Jest uses this as part of its `setupFilesAfterEnv` configuration: it runs the code in this module to set up the testing framework before each test file is executed.

To configure API mocking during test setup for MSW, import the server from `server.js`, and set up your testing framework as follows:

- Before all tests, start request interception.
- After each test, reset request handlers. This prevent handlers added during run-time in a test from affecting other tests.
- After all tests, stop request interception.

```
// setupTests.js

import { server } from './mocks/server.js'

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## References

* <a href="https://kentcdodds.com/blog/testing-implementation-details" target="_blank">Kent C. Dodds - Testing Implementation Details</a>
* <a href="https://kentcdodds.com/blog/stop-mocking-fetch" target="_blank">Kent C. Dodds - Stop Mocking Fetch</a>
* <a href="https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API" target="_blank">MDN Docs - Service Worker API</a>
* <a href="https://mswjs.io/docs/api" target="_blank">Mock Service Worker</a>
