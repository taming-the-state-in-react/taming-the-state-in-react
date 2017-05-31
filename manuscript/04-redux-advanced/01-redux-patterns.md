# Redux Patterns

## Using JavaScript ES6

- action crrators
- reducers
- etc.
- thunks get easier to read

## Naming Conventions

- like you would maybe name callback functions in your components with prefix onFoo
- actions etc.

## Nested Combined Reducers

https://stackoverflow.com/questions/36786244/nested-redux-reducers

## Command vs. Event Pattern

- https://hackernoon.com/dispatch-redux-actions-as-events-not-commands-4de8a92b1ea5

- single vs multiple reducers
- to close to command should be evaluated as local state
- Action to Reducer like 1:N

Last but not least, I want to give you clarification on the relationship between actions and reducers. You know that your application can have multiple actions and reducers. But how do they relate to each other?

- showcase one action that is captured by multiple reducers
- that will be explained in greater detail in the Command and Event patterns in Redux

## State Keys

- makes more sense when command pattern

## Folder Organization

- technical, feature
- top level when event pattern, feature level when command pattern

### Ducks

- as long as action and reducer are coupled it makes sense
- but they shouldn't be coupled. command pattern should be avoided and state keys are only useful for certian scenarios
