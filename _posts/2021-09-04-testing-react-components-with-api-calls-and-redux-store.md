## Scenario

You have a React component that makes an API call and loads the returned data into a Redux store, and you want to test the component using React Testing Library.

## Constraints

During testing, we don't want to hit the API. We might exceed API limits or, in the case of POST requests, we might edit data in the API. So, we have to mock the API requests.

Moreover, fetching data from an API is asynchronous logic. In Redux, the standard is to place asynchronous logic inside thunks. So, API calls are made inside thunks.

## How do we test thunks? How do we properly mock asynchronous requests?

Thunks are an implementation detail. (What is ID)

In React, testing is meant to test that your components work as they should for your users.

One of the guiding principles of the React Testing Library is actually that tests should be user-centric - tests should use components the way users would use the application.

As such, implementation details are not to be tested in isolation.

This prevents your tests from breaking when you refactor your components (whereby implementation details are changed, but functionality remains the same), and also prevents false positives (whereby your tests pass even though your components are not working as they should).

## So, if we do not test thunks, how do we test asynchronous logic, such as API requests, located inside of thunks?

Ideally, we want the test to still follow thunk behaviour, i.e. the asynchronous logic inside the thunk must be able to interact with the Redux store's `dispatch` and `getState` methods.

Here's what Redux recommends:

- Cover the thunk by testing the component / group of components using it.

- Mock asynchronous requests at the network level.

Mocking requests at this level makes it so that thunk logic does not have to change in the test.

The thunk will still try to make a real asynchronous request, which then gets intercepted.