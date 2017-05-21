## Advanced Reducers

- more to know

### Initial State

- similiar to initial state of store
- all reducers run a first time without any data, you can fill it with initial data

### Nested Data Structures

- state obejct not only todos but todos one property
- state instead of todo

**Nested data structures** are fine, but you want to avoid **deep nested data structures**. Another chapter will showcase this later on by using a neat helper library with the name normalizr.

### Combined Reducer

- so far you have learned that there are multiple reducers, but all the previous examples used only one reducer (TODO check if this is still true)
-the reducer can care about multiple actions though, but how to use multiple reducers?

- not only one reducer, but multuple reducers that are used in the store
- combine helper

- when to grow reducer by splitting and when to use multple reducers

### Nested Reducers

- TODO first check if that is not already done by extracting functuons
- Reducers can be nested
- my personal opinon: it is good to know that these cna be nested, but I didn't see is as widely adopted in React and Redux applications. it can make the rreasoning about the state updates more diffictul again

### Action to Reducer like 1:N

- showcase one action that is captured by multiple reducers
- that will be explained in greater detail in the Command and Event patterns in Redux

## Hands On: Redux Standalone Advanced

- combined reducer
- state instead of todoes

- nested reducer ?