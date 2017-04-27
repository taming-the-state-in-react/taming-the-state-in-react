# Redux

- one of multiple solutions
- alternative: mobx, dive into later
- successor of Flux, dan abramox and ...
- idea of unidirectional data flow
- time travelling as problem to solve

## Redux from Scratch

## Basics in Redux

- plain JS environemnt

## Redux in React

- react-redux
- connect HOC
- container (or connected components) as gateways to state

## Do I need ImmutableJs?

## Selectors

- plain selectors
- reselect with memoize

## Normalized State

- normalizr

## Command vs. Event Pattern

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

