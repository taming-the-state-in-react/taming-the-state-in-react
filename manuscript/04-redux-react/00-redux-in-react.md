# Redux in React

In the last chapters, you got to know plain Redux. It helps you to manage a predictable state object. However, you want to use this state object in an application eventually. It can be any JavaScript application that has to deal with state management. Even though Redux is a JavaScript library, the general paradigm could be applied in any programming language.

State management in single page applications (SPAs) is one of these use cases where Redux can be applied. These applications are usually built with a framework (Angular) or view layer library (React, Vue). But they lack of a sophisticated state management solution. That's where Redux comes into play. The book focuses on React, but you can apply the learnings to other solutions like Angular and Vue too.

The following scenarios could live without Redux in the first place, because they wouldn't run into state management issues with local state. But for the sake of demonstrating Redux in React, they will omit local state management and apply sophisticated state management with Redux.

## Connecting the State

On the one hand you have React as your view layer. It has everything you need to build a component hierarchy. You can compose components into each other. In addition, the component's methods make sure that you always have a hook into their lifecycle.

On the other hand you have Redux. By now, you should know how to manage state in Redux. It has basically two steps: setup and usage. First, you initialize everything by setting up reducer(s) and optional action creators. The (combined) reducer is used to create the Redux store. Second, you can interact with the store by dispatching actions, subscribing to the store and getting the current state from the store.

These three interactions from the second step need to be accessed from your view layer. As mentioned, the view layer can be anything, but to keep it focused it will be React in this book.

If you recall the unidirectional data flow in Redux, that was adapated from the Flux architecture, you will notice that you have all parts at your disposal by now.

{title="Code Playground",lang="javascript"}
~~~~~~~~
View -> Action -> Reducer(s) -> Store -> View
~~~~~~~~

How can `dispatch()`, `subscribe()` and `getState()` be accessed in React? Basically the view layer has to be able to dispatch actions on the one end, while it has to listen to updates from the store, in order to update itself, on the other end. All three functionalities are accessible on the Redux store.

## Hands On: Bootstrap React App with Redux

I can recommend to use create-react-app to bootstrap your React application. But it is up to you. If you use create-react-app and never used it before, you have to install it from the command line:

{title="Command Line",lang="text"}
~~~~~~~~
npm install -g create-react-app
~~~~~~~~

Now you can bootstrap your React application, navigate in it, and start it:

{title="Command Line",lang="text"}
~~~~~~~~
create-react-app taming-the-state-todo-app
cd taming-the-state-todo-app
npm start
~~~~~~~~

If you haven't used create-react-app before, I recommend you to read up the basics in the [official documentation](https://github.com/facebookincubator/create-react-app). Your *src/* folder has several files. You will not use the *src/App.js* file in this application, but only the *src/index.js* file. Open up your editor and adjust your *src/index.js* file to the following.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

function TodoApp() {
  return <div>Todo App</div>;
}

ReactDOM.render(<TodoApp />, document.getElementById('root'));
~~~~~~~~

Now when you start your application again with `npm start`, you should see the "Todo App" component. Before you continue to build a React application now, let's hook in all of the Redux code that you have written in the previous chapters. First, install Redux in your application.

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux
~~~~~~~~

Second, reuse the Redux code from the previous chapters in your *src/index.js* file. You start in the top to import the two Redux functionalities that you have used so far next to the imports that are already there:

{title="Command Line",lang="text"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import { combineReducers, createStore } from 'redux';
# leanpub-end-insert
import './index.css';
~~~~~~~~

Now, in between of your imports and your React code, you introduce your Redux functionlities. First, the action types:

{title="Command Line",lang="text"}
~~~~~~~~
// action types

const TODO_ADD = 'TODO_ADD';
const TODO_TOGGLE = 'TODO_TOGGLE';
const FILTER_SET = 'FILTER_SET';
~~~~~~~~

Second, the reducers:

{title="Command Line",lang="text"}
~~~~~~~~
// reducers

const todos = [
  { id: '0', name: 'learn redux' },
  { id: '1', name: 'learn mobx' },
];

function todoReducer(state = todos, action) {
  switch(action.type) {
    case TODO_ADD : {
      return applyAddTodo(state, action);
    }
    case TODO_TOGGLE : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}

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

function filterReducer(state = 'SHOW_ALL', action) {
  switch(action.type) {
    case FILTER_SET : {
      return applySetFilter(state, action);
    }
    default : return state;
  }
}

function applySetFilter(state, action) {
  return action.filter;
}
~~~~~~~~

Third, the action creators:

{title="Command Line",lang="text"}
~~~~~~~~
// action creators

function doAddTodo(id, name) {
  return {
    type: TODO_ADD,
    todo: { id, name },
  };
}

function doToggleTodo(id) {
  return {
    type: TODO_TOGGLE,
    todo: { id },
  };
}

function doSetFilter(filter) {
  return {
    type: FILTER_SET,
    filter,
  };
}
~~~~~~~~

And last but not least, the creation of the store with the combined reducers:

{title="Command Line",lang="text"}
~~~~~~~~
// store

const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
});

const store = createStore(rootReducer);
~~~~~~~~

After that, your React code follows that should be already in the same file.

{title="Command Line",lang="text"}
~~~~~~~~
function TodoApp() {
  return <div>Todo App</div>;
}

ReactDOM.render(<TodoApp />, document.getElementById('root'));
~~~~~~~~

The bootstrapping is done. You have a running React application and a Redux store. The next step is to wire both together.

## Hands On: Naive Todo with React and Redux

The following will showcase a naive usage scenario of Redux in React. So far, you have only a `TodoApp` component in React. However, you want to start a component tree that can display a list of todos and gives the user the possibility to toggle these todos. Apart from the `TodoApp` component, you will have a `TodoList` component and a `TodoItem` component. The `TodoItem` shows the name of the todo and has a functionality that is used in a button to complete the todo.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function TodoApp({ todos, onToggleTodo }) {
  return <TodoList
    todos={todos}
    onToggleTodo={onToggleTodo}
  />;
}

function TodoList({ todos, onToggleTodo }) {
  return (
    <div>
      {todos.map(todo => <TodoItem
        key={todo.id}
        todo={todo}
        onToggleTodo={onToggleTodo}
      />)}
    </div>
  );
}

function TodoItem({ todo, onToggleTodo }) {
  const { name, id, completed } = todo;
  return (
    <div>
      {name}
      <button
        type="button"
        onClick={() => onToggleTodo(id)}
      >
        {completed ? "Incomplete" : "Complete"}
    </button>
    </div>
  );
}
~~~~~~~~

Notice that none of these components is aware of Redux. They simply display todos and use a callback function to toggle todo items. Now, in the last step, you wire together Redux and React. You can use the initialized `store` in your root component where React hooks into HTML.

{title="Code Playground",lang="javascript"}
~~~~~~~~
ReactDOM.render(
  <TodoApp
    todos={store.getState().todoState}
    onToggleTodo={id => store.dispatch(doToggleTodo(id))}
  />,
  document.getElementById('root')
);
~~~~~~~~

The store does two things: it makes state accessible and exposes functionalities to alter the state. The `todos` props are passed down to the `TodoApp` by retrieving them from the `store`. In addition, a `onToggleTodo` property is passed down that is a function. This function is a higher order function that wraps the dispatching of an action that is created by its action creator. The `TodoApp` component is completly unaware of the `todos` being retrieved from the Redux store or of the `onToggleTodo()` being a dispatched action on the Redux store. These passed properties are simple props for the `TodoApp`. You can start your application again with `npm start`. The todos should be displayed, but not updated yet.

What about the update mechanism? When an action is dispatched, someone needs to subscribe to the Redux store. In a naive approach you can do the following. First, wrap your React root into a function.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function render() {
# leanpub-end-insert
  ReactDOM.render(
    <TodoApp
      todos={store.getState().todoState}
      onToggleTodo={id => store.dispatch(doToggleTodo(id))}
    />,
    document.getElementById('root')
  );
# leanpub-start-insert
}
# leanpub-end-insert
~~~~~~~~

Second, you can pass the function to the `subscribe()` method of the Redux store. Last but not least, you have to invoke the function one time for the initial render of your component tree.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function render() {
  ReactDOM.render(
    <TodoApp
      todos={store.getState().todoState}
      onToggleTodo={id => store.dispatch(doToggleTodo(id))}
    />,
    document.getElementById('root')
  );
}

# leanpub-start-insert
store.subscribe(render);
render();
# leanpub-end-insert
~~~~~~~~

The Todo application should display the todos and update the completed state once you toggle it. The final application of this approach can be found in a [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/1.0.1).

The approach showcased how you can wire up your React component tree with the Redux store. The components don't need to be aware of the Redux store at all, but the root component is. In addition, everything is re-rendered when the global state in the Redux store updates.

Even though the previous approach is pragmatic and shows a simplified version of how to wire up all these things, it is naive. Why is that? In a real application you want to avoid the following practices:

* re-render the whole component tree: You want to re-render only the components that are affected by the global state updated. Otherwise you will run into performance issues in a scaling application.

* using the store instance: You want to avoid to operate directly on the Redux store instance. The store should be injected somehow into your component tree to make it accessible for components that need to have access to the store.

* making the store globally available: The store shouldn't be globally accessible by every component. In the previous example only the React root component uses it, but who prevents you from using it directly in your `TodoItem` component to dispatch an action?

Fortunately there exists a library that takes care about these things and gives you a bridge from the Redux to the React world.

## Connecting the State, but Sophisticated

A library called [react-redux](https://github.com/reactjs/react-redux) gives you two things in order to wire up Redux with React.

First, it gives you a `<Provider />` component. When using Redux with React, the Provider component should be the root component of your application. The component gets one property as input: the Redux store that you created once with `createStore()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { Provider } from 'react-redux'

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
~~~~~~~~

After you have done this, every child component in the whole component tree has an implicit access to the store. Thus every component is able to dispatch actions and to listen to updates in order to re-render. But not every component has to. How does this work without passing the store as property to each child component? It uses the React context API. You can read more about it in the [official React documentation](https://facebook.github.io/react/docs/context.html), but just to quote the documentation for now:

*"In some cases, you want to pass data through the component tree without having to pass the props down manually at every level. You can do this directly in React with the powerful "context" API."*

That was part one to use Redux in React. Second, you can use a higher component that is called `connect` from the react-redux library. It makes the Redux store functionality dispatch and the state from the store itself available to the enhanced component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { connect } from 'react-redux'

function Component(props) {
  ...
}

const ConnectedComponent = connect(...)(Component);
~~~~~~~~

The `connect` HOC can have up to four arguments as configuration:

{title="Code Playground",lang="javascript"}
~~~~~~~~
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])(...);
~~~~~~~~

Usually you will only use two of them: `mapStateToProps()` and `mapDispatchToProps()`. You will learn about the other two arguments, `mergeProps`and `options`, later in this book.

**mapStateToProps(state, [props]) => derivedProps:** It is a function that can be passed to the connect HOC. If it is passed, the input component of the connect HOC will subscribe to updates from the Redux store. Thus it means that every time the store subscription notices an update, the `mapStateToProps()` function will run. The `mapStateToProps()` function itself has two arguments in its function signature: the global state object and optionally the props from the parent component. The function returns an object that is derived from the global state and optionally from the props from the parent component. The returned object will be merged into the remaining props that come as input in the `ConnectedComponent` component when it is used.

**mapDispatchToProps(dispatch, [props]):** It is a function (or object) that can be passed to the connect HOC. Whereas `mapStateToProps()` gives access to the global state, `mapDispatchToProps()` gives access to the dispatch method. It makes it possible to dispatch actions but passes down only plain functions that wire up the dispatching in a higher order function. After all, it makes it possible to pass functions down to the input component of the connect HOC to alter the state. You can use optionally the incoming props to wrap those into the dispatched action.

That is a lot of knowledge to digest. Both functions, `mapStateToProps()` and `mapDispatchToProps()`, can be intimidating in the beginning. In addition, they are used in a foreign higher order component. However, they only give you access to the state and to the dispatch method of the store.

{title="Code Playground",lang="javascript"}
~~~~~~~~
View -> (mapDispatchToProps) -> Action -> Reducer(s) -> Store -> (mapStateToProps) -> View
~~~~~~~~

You will see in the following examples that these functions don't need to be intimidating at all.

## Hands On: Sophisticated Todo with React and Redux

Now you will use react-redux to wire up React with Redux. Let's open up again your Todo Application in the editor. First, you have to install the new library:

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save react-redux
~~~~~~~~

Second, instead of wrapping the React root component into the `render()` function and subscribing it to the `store.subscribe()` method, you will use the plain React root component again but use the Provider component by react-redux.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { combineReducers, createStore } from 'redux';
# leanpub-start-insert
import { Provider } from 'react-redux';
# leanpub-end-insert
import './index.css';

...

# leanpub-start-insert
ReactDOM.render(
  <Provider store={store}>
    <TodoApp />
  </Provider>,
  document.getElementById('root')
);
# leanpub-end-insert
~~~~~~~~

It uses the plain `TodoApp` component. The component still expects the `todos` and `onToggleTodo` as props. But it hasn't these props. Let's use the `connect` higher order component to expose these to the `TodoApp` component. The `TodoApp` component will become a `ConnectedTodoApp` component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { combineReducers, createStore } from 'redux';
# leanpub-start-insert
import { Provider, connect } from 'react-redux';
# leanpub-end-insert
import './index.css';

...

# leanpub-start-insert
const ConnectedTodoApp = connect(mapStateToProps, mapDispatchToProps)(TodoApp);

ReactDOM.render(
  <Provider store={store}>
    <ConnectedTodoApp />
  </Provider>,
  document.getElementById('root')
);
# leanpub-end-insert
~~~~~~~~

Now, only the connections, `mapStateToProps()` and `mapDispatchToProps()` are missing. They are quite similar to the naive React with Redux version.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function mapStateToProps(state) {
  return {
    todos: state.todoState,
  };
}

function mapDispatchToProps(dispatch) {
  return {
     onToggleTodo: id => dispatch(doToggleTodo(id)),
  };
}
# leanpub-end-insert

const ConnectedTodoApp = connect(mapStateToProps, mapDispatchToProps)(TodoApp);

...
~~~~~~~~

That's it. In `mapStateToProps()` only a substate is returned. In `mapDispatchToProps()` only a higher order function that encapsulates the dispatching of an action is returned. The child components are unaware of any state or actions. They are only receiving props. The final application of this approach can be found in a [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/2.0.0).. I would advice you to compare it again with the naive version that wires React and Redux together. It is not that different from it.

## Hands On: Connecting State Everywhere

There is one last clue to understand the basics of wiring React and Redux together. In the previous example you only used one connected component that is located at the root of your component tree. But you can use connected components everywhere.

Only your `TodoApp` component has access to the state and enables you to alter the state. Instead of using your root component to connect to it to the store, you can add connected components in between. For instance, the `onToggleTodo()` function has to pass several component until it reaches its destination in the `TodoItem` component. Why not connecting the `TodoItem` component to make the functionality right next to it available rather than passing it down multiple components? The same applies for the `TodoList` component. It could be connected to retrieve the list of todos instead of getting it from the `TodoApp` component.

In the Todo application you could keep both `mapStateToProps()` and `mapDispatchToProps()`, but you would use them somewhere else. While the `TodoApp` component doesn't need them anymore, they would be used in a `ConnectedTodoItem` and `ConnectedTodoList` component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const ConnectedTodoList = connect(mapStateToProps)(TodoList);
const ConnectedTodoItem = connect(null, mapDispatchToProps)(TodoItem);
# leanpub-end-insert

ReactDOM.render(
  <Provider store={store}>
# leanpub-start-insert
    <TodoApp />
# leanpub-end-insert
  </Provider>,
  document.getElementById('root')
);
~~~~~~~~

Now you wouldn't need to pass the `onToggleTodo()` props through the `TodoApp` component and `TodoList` component anymore. The same applies for the `todos` that don't need to get passed through the `TodoApp` component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function TodoApp() {
  return <ConnectedTodoList />;
}

function TodoList({ todos }) {
  return (
    <div>
      {todos.map(todo => <ConnectedTodoItem
        key={todo.id}
        todo={todo}
      />)}
    </div>
  );
}
# leanpub-end-insert
~~~~~~~~

The final Todo application can be found in [the GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/3.0.0).

As you can imagine by now, you can connect your state everywhere to your view layer. You can retrieve it with `mapStateToProps()` and alter it with `mapDispatchToProps()` from everywhere in your component tree. These components that add this intermediate glue between view and state are called connected components. They are a subset of the container components from the container and presenter pattern. The presenter components are still clueless and don't know if the props are derived from a Redux store, from local state or actions. They just use these props.
