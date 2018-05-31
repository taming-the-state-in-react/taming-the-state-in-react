## Basics in Redux

On the [official Redux website](http://redux.js.org) it says: *"Redux is a predictable state container for JavaScript apps."*. It can be used standalone or in connection with with libraries, like React and Angular, to manage state in JavaScript applications.

Redux adopted a handful of constraints from the Flux architecture but not all of them. It has Actions that encapsulate information for the actual state update. It has a Store to save the state, too. However, the Store is a singleton. Thus, there are not multiple Stores like there used to be in the Flux architecture. In addition, there is no single Dispatcher. Instead, Redux uses multiple Reducers. Basically, Reducers pick up the information from Actions and "reduce" the information to a new state, along with the old state, that is stored in the Store. When state in the Store is changed, the View can act on this by subscribing to the Store.

{title="Concept Playground",lang="text"}
~~~~~~~~
View -> Action -> Reducer(s) -> Store -> View
~~~~~~~~

So why is it called Redux? Because it combines the two words Reducer and Flux. The abstract picture of Redux should be imaginable now. The state doesn't live in the View anymore, it is only connected to the View. What does connected mean? It is connected on two ends, because it is part of the unidirectional data flow. One end is responsible to trigger an Action to which updates the state eventually and the second end is responsible to receive the state from the Store. Therefore, the View can update according to state changes and can trigger state changes. The View, in this case, would be React, but Redux can be used with any other library or standalone as well. After all, it is only a state management container.

### Action(s)

An action in Redux is a JavaScript object. It has a type and an optional payload. The type is often referred to as **action type**. While the type is a string literal, the payload can be anything from a string to an object.

In the beginning your playground to get to know Redux will be a Todo application. For instance, the following action in this application can be used to add a new todo item:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'TODO_ADD',
  todo: { id: '0', name: 'learn redux', completed: false },
}
~~~~~~~~

Executing an action is called **dispatching** in Redux. You can dispatch an action to alter the state in the Redux store. You only dispatch an action when you want to change the state. The dispatching of an action can be triggered in your view layer. It could be as simple as a click on a HTML button. In addition, the payload in a Redux action is not mandatory. You can define actions that have only an action type. That subject will be revisited later in the book. In the end, once an action is dispatched, it will go through all reducers in Redux.

### Reducer(s)

A reducer is the next part in the chain of the unidirectional data flow. The view dispatches an action and the action object, with action type and optional payload, will pass through all reducers. What's a reducer? A reducer is a pure function. It always produces the same output when the input stays the same. It has no side-effects, thus it is only an input/output operation. A reducer has two inputs: state and action. The state is always the whole state object from the Redux store. The action is the dispatched action with a type and an optional payload. The reducer reduces - that explains the naming - the previous state and incoming action to a new state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
(prevState, action) => newState
~~~~~~~~

Apart from the functional programming principle, namely that a reducer is a pure function without side-effects, it also embraces immutable data structures. It always returns a `newState` object without mutating the incoming `prevState` object. Thus, the following reducer, where the state of the Todo application is a list of todos, is not an allowed reducer function:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function(state, action) {
  state.push(action.todo);
  return state;
}
~~~~~~~~

It would mutate the previous state instead of returning a new state object. The following is allowed because it keeps the previous state intact:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  return state.concat(action.todo);
}
~~~~~~~~

By using the [JavaScript built-in concat functionality](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/concat), the state and thus the list of todos is concatenated to another item. The other item is the newly added todo from the action. You might wonder if this embraces immutability now. Yes it does, because `concat` always returns a new array without mutating the old array. The data structure stays immutable. You will learn more about how to keep your data structures immutable later in this book.

**But what about the action type?** Right now, only the payload is used to produce a new state but the action type is ignored. So what can you do about the action type? Basically when an action object arrives at the reducers, the action type can be evaluated. Only when a reducer cares about the action type, it will produce a new state. Otherwise, it simply returns the previous state. In JavaScript, a switch case can help to evaluate different action types. Otherwise, it returns the previous state as default.

Imagine your Todo application would have a second action that toggles a Todo to either completed or incomplete. The only information which is needed as payload is an identifier to identify the Todo in the state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'TODO_TOGGLE',
  todo: { id: '0' },
}
~~~~~~~~

The reducer would have to act on two actions now: `TODO_ADD` and `TODO_TOGGLE`. By using a switch case statement, you can branch into different cases. If there is not such a case, you return the unchanged state by default.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  switch(action.type) {
    case 'TODO_ADD' : {
      // do something and return new state
    }
    case 'TODO_TOGGLE' : {
      // do something and return new state
    }
    default : return state;
  }
}
~~~~~~~~

The book already discussed the `TODO_ADD` action type and its functionality. It simply concats a new todo item to the previous list of todo items. But what about the `TODO_TOGGLE` functionality?

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  switch(action.type) {
    case 'TODO_ADD' : {
# leanpub-start-insert
      return state.concat(action.todo);
# leanpub-end-insert
    }
    case 'TODO_TOGGLE' : {
# leanpub-start-insert
      return state.map(todo =>
        todo.id === action.todo.id
          ? Object.assign({}, todo, { completed: !todo.completed })
          : todo
      );
# leanpub-end-insert
    }
    default : return state;
  }
}
~~~~~~~~

In the example, the built-in JavaScript functionality `map` is used to map over the state, the list of todos, to either return the intact todo or return the toggled todo. The toggled todo is identified by its `id` property. The [JavaScript built-in functionality map](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/map) always returns a new array. It doesn't mutate the previous state and thus the state of todos stays immutable and can be returned as a new state.

But isn't the toggled todo mutated? No, because `Object.assign()` returns a new object without mutating the old object. `Object.assign()` merges all given objects from the former to the latter into each other. If a former object shares the same property as a latter object, the property of the latter object will be used. Thus, the `completed` property of the updated todo item will be the negated state of the old todo item.

Note that these functionalities, actions and reducer, are plain JavaScript. There is no function from the Redux library involved so far. There is no hidden library magic. It is plain JavaScript with functional programming principles in mind.

There is one useful thing to know about the current reducer: It has grown in size that makes it less maintainable. In order to keep reducers tidy, often the different switch case branches are extracted as pure functions:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  switch(action.type) {
    case 'TODO_ADD' : {
# leanpub-start-insert
      return applyAddTodo(state, action);
# leanpub-end-insert
    }
    case 'TODO_TOGGLE' : {
# leanpub-start-insert
      return applyToggleTodo(state, action);
# leanpub-end-insert
    }
    default : return state;
  }
}

# leanpub-start-insert
function applyAddTodo(state, action) {
  return state.concat(action.todo);
}

function applyToggleTodo(state, action) {
  return state.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );
}
# leanpub-end-insert
~~~~~~~~

In the end, the Todo application has two actions and one reducer by now. One last part in the Redux setup is missing: the Store.

### Store

So far, the Todo application has a way to trigger state updates (actions) and a way to reduce the previous state and action to a new state (reducer). But no one is responsible to glue these parts together.

* Who delegates the actions to the reducer?
* Who triggers actions?
* And finally: Where do I get the updated state to glue it to my View?

It is the Redux store. The store holds one global state object. There are no multiple stores and no multiple states. The store is only one instance in your application. In addition, it is the first library dependency you encounter when using Redux. Therefore, use the import statement to get the functionality to create the `store` object from the Redux library.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { createStore } from 'redux';
~~~~~~~~

Now you can use it to create a store singleton instance. The `createStore` function takes one mandatory argument: a reducer. You already defined a reducer in the Reducer chapter which adds and completes todo items.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(reducer);
~~~~~~~~

In addition, the `createStore` takes a second optional argument: the initial state. In the case of the Todo application, the reducer operated on a list of todos as state. The list of todo items should be initialized as an empty array or pre-filled array with todos. If it wasn't initialized, the reducer would fail because it would operate on an `undefined` argument.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(reducer, []);
~~~~~~~~

In another chapter, the book will showcase another way to initialize state in Redux. Then you will use the reducer instead of the store to initialize state on a more fine-grained level.

Now you have a store instance that knows about the reducer. The Redux setup is done. However, the essential part is missing: You want to interact with the store. You want to dispatch actions to alter the state, get the state from the store and listen to updates of the state in the store.

So first, how to dispatch an action?

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch({
  type: 'TODO_ADD',
  todo: { id: '0', name: 'learn redux', completed: false },
});
~~~~~~~~

Second: How to get the global state from the store?

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.getState();
~~~~~~~~

And third, how to subscribe (and unsubscribe) to the store in order to listen (and unlisten) for updates?

{title="Code Playground",lang="javascript"}
~~~~~~~~
const unsubscribe = store.subscribe(() => {
  console.log(store.getState());
});

unsubscribe();
~~~~~~~~

That's all to it. The Redux store has only a slim API to access the state, update it and listen for updates. It is one of the essential constraints which made Redux so successful.

### Hands On: Redux Standalone

You know about all the basics in Redux now. A view dispatches an action on the store, the action passes all reducers and gets reduced by reducers that care about it. The store saves the new state object. Finally, a listener updates the view with the new state.

{title="Concept Playground",lang="text"}
~~~~~~~~
View -> Action -> Reducer(s) -> Store -> View
~~~~~~~~

Let's apply these learnings. You can either use your own project where you have JavaScript, JavaScript ES6 features enabled and Redux, or you can open up the following JS Bin: [Redux Playground](https://jsbin.com/zukogaj/2/edit?html,js,console). Now you are going to apply your learnings about actions, reducers, and the store from the last chapter. First, you can define your reducer that deals with adding and toggling todo items:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  switch(action.type) {
    case 'TODO_ADD' : {
      return applyAddTodo(state, action);
    }
    case 'TODO_TOGGLE' : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}

function applyAddTodo(state, action) {
  return state.concat(action.todo);
}

function applyToggleTodo(state, action) {
  return state.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );
}
~~~~~~~~

Second, you can initialize the Redux store that uses the reducer and an initial state. In the JS Bin you have Redux available as global variable.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = Redux.createStore(reducer, []);
~~~~~~~~

If you are in your own project, you might be able to import the `createStore` from the Redux library:

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { createStore } from 'redux';

const store = createStore(reducer, []);
~~~~~~~~

Third, you can dispatch your first action on the store.

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch({
  type: 'TODO_ADD',
  todo: { id: '0', name: 'learn redux', completed: false },
});
~~~~~~~~

That's it. You have set up all parts of Redux and interacted with it by using an action. You can retrieve the state by getting it from the store now.

{title="Code Playground",lang="javascript"}
~~~~~~~~
console.log(store.getState());
~~~~~~~~

But rather than outputting it manually, you can subscribe a callback function to the store to output the latest state after it has changed. Make sure to subscribe to the store before dispatching your actions in order to get the output.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const unsubscribe = store.subscribe(() => {
  console.log(store.getState());
});
~~~~~~~~

Now, whenever you dispatch an action, after the state got updated, the store subscription should become active by outputting your current state. Don't forget to unsubscribe eventually to avoid memory leaks.

{title="Code Playground",lang="javascript"}
~~~~~~~~
unsubscribe();
~~~~~~~~

A finished application can be found [in this JS Bin](https://jsbin.com/kopohur/28/edit?html,js,console). Before you continue to read, you should experiment with the project. What you see in the project is plain JavaScript with a Redux store. You can come up with more actions and deal with them in your reducer. The application should make you aware that Redux is only a state container. The state can be altered by using actions. The reducer take care of the action. It uses the action and the old state to create a new state in the Redux store.

Later you will learn about how to to connect the Redux store to your React view layer. But before doing so, let's dive into actions and reducers a bit deeper.