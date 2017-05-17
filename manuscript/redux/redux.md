# Redux

- one of multiple solutions
- alternative: mobx, dive into later
- successor of Flux, dan abramox and ...
- idea of unidirectional data flow
- time travelling as problem to solve
- Dan Abramov

## Redux from Scratch

- read up several articles that already made this

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

## Do I need ImmutableJs?

## Selectors

- plain selectors
- reselect with memoize

## Normalized State

- normalizr

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

### Redux Thunk

- basic applications
- principle of fat thunks

### Redux Saga

- mature applications

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


