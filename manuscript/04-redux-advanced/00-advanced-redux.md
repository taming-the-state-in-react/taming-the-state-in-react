# Redux, but Advanced

You have learned about Redux standalone and Redux in React. You already would be able to build applications with it. Before you dive deeper into Redux, I recommend you to experiment with your recent learnings and apply them on smaller applications. The following chapter guides you through more advanced topics in Redux. You will meet topics such as the middleware in Redux and get answers on how to keep your state immutable. It will show you to normalize your state and how to retrieve derived state by using selectors. Last but not least, you will learn about asynchronous actions. These actions will enable you to make asynchronous calls, retrieve data from an external REST API, and save it to your global state object.

## Middleware in Redux

Redux offers you to use its middleware. Every dispatched action in Redux flows through its middleware. You can opt-in a feature in between dispatching an action and the moment it reaches the reducer.

There are useful libraries out there to opt-in as features into your Redux middleware. In the following, you will get to know one of them: [redux-logger](https://github.com/evgenyrodionov/redux-logger). When you use it, it doesn't change anything in your application in production. But it will make your life easier as developer who deals with Redux. What does it do? It simply logs the actions in your developer console with `console.log()`. As a developer, it gives you clearity on which action is dispatched and how the previous and the new state are structured.

But where to apply the middleware? The `createStore()` functionality from Redux takes as third argument an enhancer. Redux comes with one of these enhancers: `applyMiddleware()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { applyMiddleware, createStore } from 'redux';

const store = createStore(
  reducer,
  undefined,
  applyMiddleware(...)
);
~~~~~~~~

Now, when using redux-logger, you can pass a `logger` instance to the `applyMiddleware()` function.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { applyMiddleware, createStore } from 'redux';
# leanpub-start-insert
import { createLogger } from 'redux-logger'

const logger = createLogger();
# leanpub-end-insert

const store = createStore(
  reducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(logger)
# leanpub-end-insert
);
~~~~~~~~

That's it. Now every action should be visible in your developer console when dispatching them. You don't have to take care about this anymore. Your state changes become more predictable as developer.

The `applyMiddleware()` functionality takes any number of middleware: `applyMiddleware(firstMiddleware, secondMiddleware, ...);`. The action will flow through all middleware before it reaches the reducers. Sometimes you have to make sure to apply them in the correct order. For instance, the `redux-logger` middleware must be last in the middleware chain in order to output the correct actions.

Nevertheless, that's only the redux-logger middleware. On your way to implement Redux applications, you will surely find out more about useful features that can be used in a middleware. Most of them are already taken care of. For instance, asynchronous actions are possible by using the Redux middleware. These will be described later in this book.

## Immutable State

Redux embraces an immutable state. Your reducers will always return a new state object. You will never mutate the state. Therefore you might have to get used to different JavaScript functionalities and syntax to embrace immutable data structures.

So far you have used built-in JavaScript functionalities to keep your data structures immutable. Such as

* array.map()
* array.concat(item)

for arrays or `Object.assign()` for objects. All of these functionalities return new instances of arrays or objects. Often you have to read the official JavaScript documentation to make sure that they return a new instance. Otherwise you would violate the constraints of Redux.

But it doesn't end here. You should know about your tools to keep data structures immutable. There are a handful of third-party libraries that can support you in keeping them immutable.

* [immutable.js](https://github.com/facebook/immutable-js)
* [mori.js](https://github.com/swannodette/mori)
* [seamless-immutable.js](https://github.com/rtfeldman/seamless-immutable)
* [baobab.js](https://github.com/Yomguithereal/baobab)

But they come with three drawbacks. First, they add another layer of complexity to your application. Second, you have to learn yet another library. And third, you have to hydrate your data most of the time, because most of these libraries wrap your vanilla JavaScript objects and arrays into a library specific immutable data object/array. It is an immutable data object/array in your Redux store, but once you want to use it in React you have to transform it into a plain JavaScript object/array.

Personally I would recommend to use such libraries only in two scenarios:

* You are not comfortable to keep your data structures immutable with JavaScript ES5, JavaScript ES6 and beyond.
* You want to improve the performance of immutable data structures when using huge amounts of data.

If both statements are false, I would advice you to stick to plain JavaScript. As you have seen the built-in JavaScript functionalities already help a lot. In JavaScript ES6 and beyond you get one more functionality to keep your data structures immutable: [spread operators](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator). Spreading an object or array into a new object or new array always gives you a new object or new array.

Do you recall your reducer that handles the todos from your Todo application?

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
  const todo = Object.assign({}, action.todo, { completed: false });
  return state.concat(todo);
}

function applyToggleTodo(state, action) {
  return state.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );
}
~~~~~~~~

You can express them with JavaScript ES6 too.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
# leanpub-start-insert
  const todo = { ...action.todo, completed: false };
  return [ ...state, todo ];
# leanpub-end-insert
}

function applyToggleTodo(state, action) {
  return state.map(todo =>
    todo.id === action.todo.id
# leanpub-start-insert
      ? { ...todo, completed: !todo.completed }
      : todo
# leanpub-end-insert
  );
}
~~~~~~~~

JavaScript gives you enough tools to keep your data structures immutable. THere is no need to use third-party library except for the two mentioned use cases. However, there might be a third use case where such library would help: deeply nested data structures in Redux that need to be kept immutable. It is true that it becomes more diffictult to keep data structures immutable with each nested level. But, as mentioned ealier in the book, it is bad practice to have deeply nested data structures in Redux in the first place. That's were the next chapter of the book comes into play.

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
