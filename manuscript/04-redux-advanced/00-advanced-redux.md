# Redux, but Advanced

## Middleware in Redux

- log
- async actions later

## Immutable State

- so far Object.assign
- ES6!
http://redux.js.org/docs/recipes/UsingObjectSpreadOperator.html

- other libraties immutableJs, moba?
- https://medium.com/@fastphrase/should-i-use-immutablejs-with-redux-58f88d6fd81e
- two ways: beginner and large scaling applications can use helper libs
- my recommendation is to stay to ES6 only
- problems like: you would run into state hydrations: rehydrating and dehydrating

- add reference list pf examples

## Normalized State

- normalizr
- https://www.reddit.com/r/reactjs/comments/5tnk9e/react_redux_how_to_use_normalized_state/

## Selectors

- plain selectors
- reselect with memoize

## Asynchronous Actions

- so far only synchronrous actions
- https://decembersoft.com/posts/what-is-the-right-way-to-do-asynchronous-operations-in-redux/

### Redux Thunk

- basic applications
- principle of fat thunks

### Redux Saga

- mature applications

### Redux Observable

- comparison to Saga http://stackoverflow.com/questions/40021344/why-use-redux-observable-over-redux-saga

### Redux Cycle

- valid alternative for reactive programming
