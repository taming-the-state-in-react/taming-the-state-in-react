# Redux Ecosystem Outline

After learning the basics and advanced techniques in Redux and applying them on your own in an application, you are ready to explore the Redux ecosystem. The Redux ecosystem is huge and cannot be covered in one book. However, this chapter attempts to outline different paths you can take to explore the world of Redux. Apart from outlining these different paths, a couple of topics will be revisited as well to give you a richer toolset when using Redux.

Before you are left alone with the last chapter covering Redux, I want to make you aware of [this repository](https://github.com/markerikson/redux-ecosystem-links) by Mark Erikson. It is a categorized list of Redux related add-ons, libraries and articles. If you get stuck at some point, want to find a solution for your problem, or are just curious about the ecosystem, check out the repository. Otherwise, I encourage you to join the official [Slack Group](https://slack-the-road-to-learn-react.wieruch.com/) for further recommendations.

## Redux DevTools

The Redux DevTools are essential for many developers when implementing Redux applications. It improves the Redux development workflow by offering a rich set of features such as inspecting the state and action payload, time traveling and realtime optimizations.

How does it work? Basically, you have two choices to use the Redux DevTools. Either you integrate it directly into your project by using its node package with npm or you install the official browser extension. While the former comes with an implementation setup in your application, the latter can simply be installed for your browser without changing your implementation.

The most obvious feature is to inspect actions and state. Rather than using the [redux-logger](https://github.com/evgenyrodionov/redux-logger), you can use the Redux DevTools to get insights into these information. You can follow each state change by inspecting the action and the state.

Another great feature is the possibility to time travel. In Redux you dispatch actions and travel from one state to another state. The Redux DevTools enable you to travel back in time by reverting actions. For instance, that way you wouldn't need to reload your browser anymore to follow a set of actions to get to a specific application state. You could simply alter the actions in between by using the Redux DevTools. You can trace back what action led to which state.

In addition, you can persist your Redux state when doing page reloads with Redux DevTools. That way, you don't need to perform all the necessary actions to get to a specific state anymore. You simply reload the page and keep the same application state. This enables you to debug your application when having one specific application state.

However, there are more neat features that you might enjoy while developing a Redux application. You can find all information about the Redux DevTools in the [official repository](https://github.com/gaearon/redux-devtools).

## Connect Revisited

In one of the previous chapters, you have connected your view layer to your state layer with [react-redux](https://github.com/reactjs/react-redux). There you have used the provider pattern in React to make the state accessible to your entire view layer.

The `connect` higher-order components enabled you to wire the Redux store to any component. The most often used two arguments are `mapStateToProps()` and `mapDispatchToProps()` for the connect higher-order component. While the former gives access to the state, the latter gives access to actions to be dispatched for manipulating the state.

However, `connect` has two more optional arguments that shouldn't stay uncovered in this book.

The third argument is called `mergeProps()`. As arguments it gets the result from `mapStateToProps()`, `mapDispatchToProps()` and the parent props: `mergeProps(stateProps, dispatchProps, ownProps)`. The function returns props as an object to the wrapped component. Basically, it gives you an intermediate layer to mix up `stateProps` and `dispatchProps` before they reach the wrapped component. However, it is rarely used. Often, when mixing up state and actions in this layer, it is associated with a bad state architecture. You should ask yourself if something else can be changed to avoid this intermediate layer.

The fourth argument is called `options`. It is an object to configure the connect higher-order component. It comes with these additional properties: `pure`, `areStatesEqual()`, `areOwnPropsEqual()`, `areMergedPropsEqual()`. How does it work altogether? When the first argument, the `pure` property, is set to true, the connect higher-order component will avoid re-rendering the view and avoids the calls to its arguments `mapStateToProps()`, `mapDispatchToProps()` and `mergeProps()`. But only when the equality checks of `areStatesEqual()`, `areOwnPropsEqual()`, `areMergedPropsEqual()` remain equal based on their respective equality checks. These equality checks are performed on the previous state and props and updated state and props. These equality checks can be modified in the options `areStatesEqual`, `areOwnPropsEqual`, `areMergedPropsEqual`. Otherwise they have a default equality check.

After all, the `options` are a pure performance optimization. It is not often used when developing Redux applications. Basically, you can set the `pure` property to true to avoid re-renderings and other argument evaluations of the `connect` higher-order component. But it comes with certain default equality checks that can be configured. In addition, the underlying assumption is that the wrapped component is a pure component and doesn't rely on any other side-effect data.

If you want to read up the `connect` higher-order component again, you can checkout the [official repository of react-redux](https://github.com/reactjs/react-redux) and look for the `connect` chapter.

## Concise Actions and Reducers

Redux made state management predictable with clear constraints. Yet, these constraints come with a lot of code to manage actions and reducers. There are people who argue that writing Redux code is verbose. That's why there exist utility libraries on top of Redux to reduce the boilerplate code. One of them is called [redux-actions](https://github.com/acdlite/redux-actions).

The library attempts to make your actions and reducers concise. It comes with three methods: `createAction()`, `handleAction()` and `combineActions()`. The book will give you a brief overview of the former two methods.

The `createAction()` method is a utility for action creators. To be more specific, the method should be named: `createActionCreator()`. The only required argument for the method is an action type.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { createAction } from 'redux-actions';

const doAddTodo = createAction('TODO_ADD');
~~~~~~~~

The `doAddTodo()` is an action creator. It uses the specified action type 'TODO_ADD'. When using it, you can pass a payload when needed. It becomes automatically allocated under a payload property.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const action = doAddTodo({ id: '0', name: 'learn redux', completed: false });

// action: {
//   type: 'TODO_ADD',
//   payload: {
//     id: '0',
//     name: 'learn redux',
//     completed: false
//   }
// }
~~~~~~~~

The `handleAction()` method is a utility for reducers. It aligns action types with reducers whereas no switch case statement is needed anymore. It takes the action type as argument and a reducer function for handling the incoming action. As third argument, it takes an initial state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { handleAction } from 'redux-actions';

handleAction('TODO_ADD', applyAddTodo, {});

function applyAddTodo(state, action) {
  // ...
  // return new state
}
~~~~~~~~

The two methods `createAction()` and `handleAction()` have sibling methods for using, creating, and handling multiple actions too: `createActions()` and `handleActions()`. Especially when defining a reducer, it makes sense to map multiple action types to multiple handlers.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { handleActions } from 'redux-actions';

const reducer = handleActions({
  TODO_ADD: applyAddTodo,
  TODO_TOGGLE: applyToggleTodo,
}, initialState);
~~~~~~~~

As you can see, it is far more concise than defining reducers in plain JavaScript with the switch case statement.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state = initialState, action) {
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
~~~~~~~~

The drawback when using the library is that it hides how Redux works with plain JavaScript. It can be difficult for newcomers to grasp what's going on when using such utility libraries from the very beginning without understanding how actions and reducers in Redux work.

The library is only a small utility belt for Redux, yet a lot of people are using it. You can read up everything about it in the [official documentation](https://github.com/acdlite/redux-actions).

## React Redux Libraries

Apart from the [react-redux](https://github.com/reactjs/react-redux) library that glues together your view and state layer, there exist other libraries that can be used when you already use React and Redux. Usually, these libraries provide you with React higher-order components that are coupled to the Redux store. That way, you don't need to worry about the state management when it is shielded away from you.

For instance, when using HTML forms in React, it is often tedious to track the state of each input element in your local component. Moreover you are often confronted with validation of these forms. The library [redux-form](https://github.com/erikras/redux-form) helps you to keep track of the form state not in the local state but in the Redux store. It enables you to access and update the form state through a higher-order component that is connected to the Redux store. In addition, it supports you in validating your form state before a user can submit it.

Another example would be a table component in React. A plain table component in React can be easily written on your own. But what about certain features such as sorting, filtering or pagination? Then it becomes difficult, because you would have to manage the state of each initialized table component. There exist several libraries that help you to implement tables in React and glue them to the Redux store. For instance, the [react-redux-composable-list](https://github.com/SmallImprovements/react-redux-composable-list) can be used for such cases.

There are a ton of libraries that already abstract away the state management for you when using common components such as forms or tables. Again you can have a look into [this repository](https://github.com/markerikson/react-redux-links) to get to know various of these libraries. It makes sense to use battle tested abstractions as libraries before implementing them on your own.

## Routing with Redux

In single page applications you will introduce routing eventually. In React, there exists one preferred library for routing: [React Router](https://github.com/ReactTraining/react-router). There should be other routing libraries for other single page application solutions. These solutions help you to navigate from URL to URL without reloading the page. That's how single page applications work after all. You only fetch your application once, but keep track of the state even when you route from URL to URL. Thus the routes in your URLs are state, too. But is it managed in the Redux store?

The common sense when using routing in Redux is that the Router handles the URL and Redux handles the state. There is no interaction between them. For instance, when you decide to store your visibility filter `SHOW_ALL` into your URL (domain.com?filter=SHOW_ALL) instead of your Redux store, it is fine doing it. You would only have to retrieve the state from the URL and not from the Redux store. So it depends on your own setup. In the end, the Router holds the single source of truth for the URL state and the Redux store holds the single source of truth for the application state. You can read more about this topic in the [official documentation](http://redux.js.org/docs/advanced/UsageWithReactRouter.html) of Redux.

## Typed Redux

JavaScript by nature is an untyped language. You will often encounter bugs in your career that could have been prevented by type safety. In Redux, type safety can make a lot of sense, because you can define exactly what kind of types go into your actions, reducers or state. You could define that an action that creates a todo item would have the property `name` with the type String and the property `completed` with the type Boolean. Every time you pass a wrong typed value for these properties to create a todo item, you would get an error on compile time of your application. You wouldn't wait until your application runs to figure out that you have passed a wrong value to your action. There wouldn't be a runtime exception when you can already cover these bugs during compile time.

Typed JavaScript can be a verbose solution when working on short living or simple projects. But when working in a large code base, where code needs to be kept maintainable, it is advisable to use a type checker. It makes refactorings easier and adds a bunch of benefits to the developer experience due to editor and IDE integrations.

There exist two major solutions gradually using JavaScript as a typed language: Flow (Facebook) and TypeScript (Microsoft). While the former has its biggest impact in the React community, the latter is well adopted amongst other frameworks and libraries.

What would a type checker like Flow look like when using in Redux? For instance, in a todo reducer the state could be defined by a type:

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
type Todo = {
  id: string,
  name: string,
  completed: boolean,
};

type Todos = Array<Todo>;

function todoReducer(state: Todos = [], action) {
# leanpub-end-insert
  switch(action.type) {
    case ADD_TODO : {
      return applyAddTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

Now, whenever an action leads to a state that is not defined by its type definition, you would get an error on compile time of your application. In addition, you can use plugins for your editor or IDE to give you the early feedback that something is wrong with your action or reducer. As the previous example has shown type safety for reducers, you could apply the same for your action creators and selectors. Everything can be type checked. You can read more about [Flow on its official site](https://flow.org/).

## Server-side Redux

Server-side rendering is used to render the initial page load from a server. Every further user interaction is done on the client-side. For instance, it is beneficial for SEO, because when a web crawler visits your website, it can retrieve the whole application without bothering to execute JavaScript on the client-side. It retrieves the whole application with its initial state. The initial state can already be data that is fetched from a database. In React, but also in other single page applications, there are solutions to deal with server-side rendering. However, introducing server-side rendering comes with a handful of challenges. One of these challenges is state management.

When the initial page is rendered by the server-side, the initial state must be sent as a response to the client as well. The client in return would use the initial state. For instance, imagine you want to load data from a database before you send the response from the server to the client. Somehow you would have to put this data into the response next to your server-side rendered application. Afterward, the client can use the response to render the application and would already have the initial state that comes from a database. If the data wasn't sent along in the initial server request, the client would have to fetch it again.

In Redux, you can initialize a Redux store anywhere. You can initialize it on a client-side to access and manipulate the state, but also on the server-side to provide your application with an initial state. The initial state would be put in the Redux store before the server-sided response is send to the client application. But how does it work? The Redux store on the client-side is a singleton. There is only one instance of the Redux store. On the server-side, the Redux store isn't a singleton. Every time a server-side request is made, it would initialize a new instance of the Redux store. The Redux store can be filled with an initial state before the server-side response is sent to a client.

Server-side rendering and state management open up a whole new topic. That's why the book doesn't cover the topic but only points you in the right direction. You can read more about the topic in the [official Redux documentation](http://redux.js.org/docs/recipes/ServerRendering.html).
