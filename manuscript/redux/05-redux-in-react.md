## Redux in React

In the last chapters you got to know plain Redux. It helps you to manage a predictable state object. However, you want to use this state object in an application eventually. It can be any JavaScript application that has to deal with state management. Even though Redux is a JavaScript library, the general paradigm could be applied in any programming language.

State management in single page applications (SPAs) is one of these use cases where Redux can be applied. These applications are usually built with a framework (Angular) or view layer library (React, Vue). The book focuses on React, but you can apply the learnings to other solutions too.

The following scenarios will omit local state management and apply sophisticated state management with Redux. Even though these examples could live without Redux, because they wouldn't run into scaling state management issues.

### Connecting State

On the one hand you have React as your view layer. It has everything you need to build a component hierarchy. You can compose components into each other. In addition, its lifecycle methods make sure that you always have a hook into their lifecycle.

On the other hand you have Redux. By now, you should know how to manage state in Redux. It has basically two steps. First, you initialize everything by setting up reducer(s) and optional action creators. The (combined) reducer is used to create the Redux store. Second, you can interact with the store by dispatching actions, subscribing to the store and getting the current state from the store.

These three interactions from the second step need to be accessed from your view layer. As mentioned, the view layer can be anything, but to keep it focused it will be React in this book.

If you recall the unidirectional data flow in Redux that was adapated from the Flux architecture, you will notice that there are all parts at your disposal by now.

{title="Code Playground",lang="javascript"}
~~~~~~~~
View -> Action -> Reducer(s) -> Store -> View
~~~~~~~~

How can `dispatch`, `subscribe` and `getState` be accessed in React? Basically the view layer has to be able to dispatch actions on the one end, while it has to listen to updates from the store in order to update itself on the other end. All three functionalities are accessible on the Redux store.

### Hands On: Todo with React and Redux

Let's open up the third time the Redux Playground. This time you will wire up React to the Redux store. The [JS Bin project from the last chapter](https://jsbin.com/kopohur/15/edit?html,js,console,output) already imports all necessary libraries: redux, react and react-dom.

- initial state are some todos

{title="Code Playground",lang="javascript"}
~~~~~~~~
const initialState = {
  todos: [
    { id: '0', name: 'learn redux' },
    { id: '1', name: 'learn mobx' },
  ]
};
const store = Redux.createStore(rootReducer, initialState);
~~~~~~~~

- then build a simple component tree

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

- now you can use the store in the root component where you have your React hook into the HTML. to get the state and to dispatch actions, the child component will not know that the `onToggleTodo` is a dispatched action and the todos are coming from the Redux store

{title="Code Playground",lang="javascript"}
~~~~~~~~
function render() {
  ReactDOM.render(
    <TodoApp
      todos={store.getState().todos}
      onToggleTodo={id => store.dispatch(doToggleTodo(id))}
    />,
    document.getElementById('root')
  );
}
~~~~~~~~

- last but not least you can subscribe to the store to render the whole component tree

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.subscribe(render);
render();
~~~~~~~~

- final: https://jsbin.com/kopohur/16/edit?html,js,console,output

- now you might ask: "Is that everything?"
- in a real applicartion
-- you want to avoid to pass around the store directy
-- you want to avpoid to make the store globally accessible (even though you only used it in the root component now)
-- you want to avoid to re-render the whole component tree for every state change

### react-redux

Fortunately you don't need to wire up the store your own. A library called [react-redux](https://github.com/reactjs/react-redux) gives you two things that you will need to make it work in React.

First, it gives you a `<Provider />` component. When using Redux with React, the Provider component should be the root component of your application. The component gets one property as input: the Redux store that you created once with `createStore()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { Provider } from 'react-redux'

<Provider store={store}>
  <App />
</Provider>
~~~~~~~~

After you have done this, every child component in the whole component tree has access to the store. Thus every component is able to dispatch actions and to listen to updates in order to re-render. How does this work without passing the store as property to each child component? It uses the React context API. You can read more about it in the [official React documentation](https://facebook.github.io/react/docs/context.html), but just to quote the documentation for now:

*"In some cases, you want to pass data through the component tree without having to pass the props down manually at every level. You can do this directly in React with the powerful "context" API."*

That was part one to use Redux in React. Second, you can use the higher component with the name `connect` that comes from `react-redux`. It makes the store functionalities dispatch and the state from the store itself available to the input component. It can be used everywhere in your component tree.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { connect } from 'react-redux'

function AnyComponent(props) {
  ...
}

const ComponentWithAccessToStore = connect(...)(AnyComponent);
~~~~~~~~

The `connect` HOC can have up to four arguments as configuration.

{title="Code Playground",lang="javascript"}
~~~~~~~~
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])(...);
~~~~~~~~

Usually you will only use two of them: `mapStateToProps` and `mapDispatchToProps`. You will learn about the other two arguments, `mergeProps`and `options`, later in this book.

**mapStateToProps(state, [props]):** It is a function that can be passed to the connect HOC. If it is passed, the input component of the connect HOC will subscribe to updates from the Redux store. Thus it means that every time the store subscription notices an update, the `mapStateToProps()` function will run. The `mapStateToProps()` function itself has two arguments in its function signature: the global state object and optionally the props from the parent component. The function returns an object that is derived from the global state and from the optional props from the parent component. The returned object will be merged into the props that are received in the input component of the connect HOC.

**mapDispatchToProps(dispatch, [props]):** It is another function that can be passed to the connect HOC. Whereas `mapStateToProps()` gives access to the global state, `mapDispatchToProps()` gives access to the dispatch method that you can use it to dispatch actions. After all, it makes it possible to pass functions down to the input component of the connect HOC to alter the state. You can use optionally the props and higher order functions to wrap those into the dispatched action.

That is a lot of knowledge to digest. Both functions, `mapStateToProps()` and `mapDispatchToProps()`, can be intimidating in the beginning. In addition, they are used in a foreign higher order component. However, they only give you access to the state and to the dispatch method of the store. You will see in the following examples that they don't need to be intimidating at all.


- JS Bin: react with todo app

- react-redux

### Managing State from Everywhere

- containers can be placed everywhere
- container (or connected components) as gateways to state

### Showing State from Everywhere

- State -> View
- presenters that only get callbacks to alter state
- don't know that they are connected to local state or Redux store
- showcase with presenter

### Hands On: Snake with React and Redux

- create-react-app