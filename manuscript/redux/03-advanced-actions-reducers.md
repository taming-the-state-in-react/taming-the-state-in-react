## Advanced Actions

- there are some more fine grained

### Minimum Action Payload

Do you recall the action that added a Todo?

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'ADD_TODO',
  todo: { id: '0', name: 'learn redux', completed: false },
}
~~~~~~~~

The payload of an action should be the minimum payload a reducer would need to alter the state in the Redux store. In the example, when you want to add a Todo in a Todo application, it would need at least the unique identifier and a name of a Todo. But the `completed` property is unneccesary. The assumption is that every Todo that is added to the store will be incompleted. It wouldn't make sense in a puristic Todo application to add completed Todos.

But who takes care about the `completed` property? The Reducer could take over.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  const { name, id } = action.todo;
  const newTodo = { name, id, completed: false };
  return [...state.todos, newTodo];
}
~~~~~~~~

That's why it is sufficient to define a minimum payload in the action.

### Action Type

- constant extracted
- can be used in both, actions and reducers
- even though they are at different places

### Action Creator

- so far all actions had hardcoded payload (TODO check if this is still true)
- at some point you want to make the payload flexible
- that's were you can use so called action creators
- they only return the action object
- substututed in action creator, but it is not mandatory to use them!
- how does it look in store dispatch

### Thoughtfully planned Actions

The book mentioned earlier that actions don't need to have a payload. Only the action type is mandatory.

For instance, imagine you want to login into your application. Therefore you need to open up a modal where you can enter your credentials: email and password. You wouldn't need a payload for your action in order to open a modal. You only need to signalize by dispatching an action that the modal state should be open.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'OPEN_LOGIN_MODAL',
}
~~~~~~~~

A reducer would take care of it and set the state of a `isLoginModalOpen` property to true. While it is good to know that the payload is not mandatory in actions, the last example can lead to bad practice. You already know that you would need a second action to close the modal again.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'CLOSE_LOGIN_MODAL',
}
~~~~~~~~

A reducer would set the `isLoginModalOpen` property in the state to false. That's verbose, because you already need two actions to alter only one property.

By planning your actions thoughtfully, you avoid these bad practices and keep your actions on a higher level of abstraction. If you would use the payload for the action, you could deal with the login scenario in only one action instead of two. The `isLoginModalOpen` would be dynamically set in the action rather than hardcoded in a Reducer.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'TOGGLE_LOGIN_MODAL',
  isLoginModalOpen: true,
}
~~~~~~~~

By using an action creator, the payload can stay flexbile.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doToggleLoginModal(open) {
  type: 'TOGGLE_LOGIN_MODAL',
  isLoginModalOpen: open,
}
~~~~~~~~

In idiomatic Redux actions should always try to stay on an abstracter level rather than on a concrete level. This will be explained more in detail in another chapter that is about Command and Event patterns in Redux. (TODO check again when other chapter is written.)

### Payload Structure

- payload object

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
- action creators

- nested reducer ?
- action more abstracted ?