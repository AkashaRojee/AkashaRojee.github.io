## Introduction

### Scenario

You have a React component that makes an API call and loads the returned data into a Redux store, and you want to test the component using React Testing Library.

## Assumptions

- Create React App is used.
- The API is a REST API.
- Jest is used as test runner.

### Constraints

During testing, we don't want to hit the API. We might exceed API limits or, in the case of POST requests, we might edit data in the API. So, we have to mock the API requests.

Moreover, fetching data from an API is asynchronous logic. In Redux, the standard is to place asynchronous logic inside thunks. So, API calls are made inside thunks.

### Questions

How do we test thunks? How do we properly mock asynchronous requests?

## Testing thunks?

Thunks are an implementation detail (something which your users don't use/see/know about).

In React, testing is meant to test that your components work as they should for your users.

One of the guiding principles of React Testing Library is actually that tests should be user-centric - tests should use components the way users would use the application.

As such, implementation details are not to be tested in isolation.

This prevents your tests from breaking when you refactor your components (whereby implementation details are changed, but functionality remains the same), and also prevents false positives (whereby your tests pass even though your components are not working as they should).

So, if we do not test thunks, how do we test asynchronous logic, such as API requests, located inside of thunks?

## Testing asynchronous logic

Ideally, we want the test to still follow thunk behaviour, i.e. the asynchronous logic inside the thunk must be able to interact with the Redux store's `dispatch` and `getState` methods.

Here's what Redux recommends:

- Cover the thunk by testing the component / group of components using it.

- Mock asynchronous requests at the network level.

## Mocking API requests

Mocking API requests at the network level makes it so that thunk logic does not have to change in the test.

The thunk will still try to make a real API request, which then gets intercepted by the mock implementation, after which the thunk proceeds normally.

**Note:** Redux recommends to use Redux Toolkit's `createAsyncThunk` to write thunks instead of writing them manually. This way, the thunk can handle promise-based action types of the API response (`pending`, `fulfilled`, `rejected`).

Since network-level API mocking involves intercepting a real API request to return a mock response, why not just mock the API request itself?

You don't expect your users to mock fetching data from an API. Going by the React Testing Library's guiding principles, your tests shouldn't mock fetching either.

At best, you could mock fetching by reimplementing the API as a function and setting this as the mock implementation of the window's fetch, using spies. However, if the API's implementation changes (e.g properties and headers of the fetch response), you'll have to change your mock function. Moreover, if there's an error with the way you call your fetch in the test, the test could still pass, but it'd actually be a broken test.

The better solution is therefore to test API calls with implementation details out of the picture.

## API mocking libraries

Enter API mocking libraries that operate at the network level.

This way, your test will make a real API request, and the mocking library will intercept it and return the mock response that you specified for the given API endpoint. You won't need to mock the implementation of the API itself, nor the API request.

Some of the tools that Redux recommends are Mock Service Worker, MirageJS, Jest Fetch Mock, and Fetch Mock. The one I use is Mock Service Worker (MSW).

MSW is an API mocking library that uses the Service Worker API to intercept actual requests. Here, the service worker acts like an intermediary server between your app and the network, and intercepts network requests from your app.

## How MSW works

1. MSW registers a service worker that listens to your app's network requests.

2. During your test, when the thunk makes the API request, the service worker intercepts the request and sends it to MSW.

3. MSW returns the mock response (specified beforehand in MSW configuration) to the service worker.

4. The service worker sends that response to your test.

### MSW How-To

1. Installation

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

2. Configure MSW

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
*For a full example of response resolvers, see https://mswjs.io/docs/getting-started/mocks/rest-api#response-resolver.*

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

3. Configure API mocking

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

https://kentcdodds.com/blog/testing-implementation-details
https://kentcdodds.com/blog/stop-mocking-fetch
https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API
https://mswjs.io/docs/api
