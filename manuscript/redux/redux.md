# Redux

- one of multiple solutions
- alternative: mobx, dive into later
- successor of Flux, dan abramox and ...
- idea of unidirectional data flow
- time travelling as problem to solve
- Dan Abramov

- TODO introduction: http://blog.isquaredsoftware.com/presentations/2017-02-react-redux-intro/#/58 or check if it deals with react-redux

- http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-1/

## Redux from Scratch

- read up several articles that already made this
- https://news.ycombinator.com/item?id=14273549
- https://zapier.com/engineering/how-to-build-redux/

## Basics in Redux

- plain JS environemnt
- functiona: pure functions,

### Reducer

### Actions

### Action Types

### Action Creator

## Store

- dispatch
- listen

# Hands On: Redux Standalone

- little hands on implemention

## Redux in React

- react-redux
- connect HOC that listen to the store
- container (or connected components) as gateways to state

## Gateway Components

- mix up presenter and container

## How to achieve Immutability?

- ES6!

- other libraties immutableJs, moba?
- https://medium.com/@fastphrase/should-i-use-immutablejs-with-redux-58f88d6fd81e
- two ways: beginner and large scaling applications can use helper libs
- my recommendation is to stay to ES6 only
- problems like: you would run into state hydrations: rehydrating and dehydrating

## Nested Reducer

- TODO

## Selectors

- plain selectors
- reselect with memoize

## Normalized State

- normalizr
- https://www.reddit.com/r/reactjs/comments/5tnk9e/react_redux_how_to_use_normalized_state/

## Command vs. Event Pattern

- single vs multiple reducers
- to close to command should be evaluated as local state

## State Keys

- makes more sense when command pattern

## Folder Organization

- technical, feature
- top level when event pattern, feature level when command pattern
### Ducks?

## State Architecture

- under the foundation of
-- folder organizatuon
-- state keys
-- comand and event pattern

- what are the domain slices
- view state (maybe local state), entitiy state
- normalization

## Asynchronous Redux

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

## Routing with Redux

- how to
- clear state?

## Typed Redux

- flow
- alternative: typescript

## Fullstack Redux

- sevrer side rendering
- sockets

# Hands On: Snake with Redux

- show off command (only one reducer cares, but refactor it to local state) vs event (multiple reducers care) pattern

# Hands On: Hacker News with Redux

- you have built one in plain React in the Road to learn React
- show off normalizr

# Hands On: Todo App with Redux

- show off routing (? maybe too much)

# Redux Forms

- naive usage 1 chapter without library
- library 2 chapter
- under the hood 3 chapter, you can do your own HOC libraries that connect to the Redux store!


