# Redux

Redux is one of the libraries that helps you implementing sophistaicated state management in your applications. It goes beyond the local state. It is one of the solutions you would take in a scaling application in order to tame the state. A React application is a perfect fit for Redux, yet other libraries and frameworks highly adopted it as well.

**Why is Redux that popular in the JavaScript community?** In order to answer that question, I have to go a bit into the past of JavaScript applications. In the beginning, there was one library to rule them all: jQuery. It was mainly used to manipulate the DOM, to amaze with animations and to implement reusbale widgets. It was the number one library in JavaScript. There was no way around it. However, the usage of jQuery skyrocketed and applications grew in size. But not in size of HTML and CSS. It was the size of code in JavaScript. Eventually the code in those applications became a mess, because there was no proper architecture around it. The infamous spaghetti code became a problem in JavaScript applications.

It was about time for noveau solutions to emerge which would go beyond jQuery. These libraries, most of them frameworks, would bring the tools for proper architectures in frontend applications. In addition, they would bring opinionated approaches to solve problems. These solutions enabled developers to implement single page applications (SPAs).

Single page applications became popular when the first generation of frameworks and libraries, among them Angular 1, Ember and Backbone, were released. Suddenly developers had frameworks to build scaling frontend applications. However, as history repeats itself, with every new technology there will be new problems. In SPAs every solution had a different approach for state management. For instance, Angular 1 used the famous two-way data binding. It embraced a bidirectional data flow. Only after applications scaled, the problem in state management became widely known.

During that time React was released by Facebook. It was among the second generation of SPA solutions. Compared to the first generation, it was a library that only leveraged the view layer. It came with an own state management solution though: local state management.

In React the principle of the unidirectional data flow became popular. State management should be more predictable in order to reason about it. Yet, the local state management wasn't sufficient at some point. React applications scaled very well, but ran into the same problems of predictable and maintainable state management. Even though the problems weren't that destructive than in bidirectional data flow applications, there was still a scaling problem. That was the time when Facebook introduced the Flux architecture.

The Flux architecture is a pattern to deal with state management in scaling applications. The official website says that *"[a] unidirectional data flow is central to the Flux pattern [...]"*. The data flows only in one direction. Apart from the unidirectional data flow, the Flux architecture came with 4 essential components: Action, Dispatcher, Store and View. The View is basically the component tree in a modern application. An user can interact with the View in order to trigger an Action. An Action would encapsulate all the necessary information to update the state in the Store(s). The Dispatcher on the way delegates the Actions to the Store(s). The updated state would be propagated to the Views again to update them.

The unidirectional goes in one direction. A View can trigger an Action, that goes through the Dispatcher and Store, and would change the View eventually when the state in the Store changed. The unidirecitonal data flow is enenclosed in this loop. Then again, a view can trigger another action. Since Facebook introduced the Flux architecture, the View was associated with React.

You can [read more about the Flux architecture](https://facebook.github.io/flux/) on the official website. There you will find a [video about its introduction at a conference](https://youtu.be/nYkdrAPrdcw?list=PLb0IAmt7-GS188xDYE-u1ShQmFFGbrk0v) too. If you are interested about the origins of Redux, I highly recommend to read and watch the material.

After all, Redux became the successor library of the Flux architecture. Even though there were a bunch of solutions around the Flux architecture, Redux managed to succeed. But why did it succeed?

[Dan Abramov](https://twitter.com/dan_abramov) and [Andrew Clark](https://twitter.com/acdlite) are the creators of Redux. It was [introduced by Dan Abramov at React Europe](https://www.youtube.com/watch?v=xsSnOQynTHs) in 2015. However, the talk by Dan doesn't introduce Redux per se. Instead the talk introduced a problem that Dan Abramov faced that led to Redux. I don't want to foreclose the content of the talk, that's why I encourage you to watch the video. If you are keen to learn Redux, you should dive into the problem that was solved by it for Dan Abramov.

Nevertheless, one year later, again at React Europe, Dan Abramov reflects about the journey of Redux and its success. He mentions a few things that made Redux successful in his opinion. The two main take aways of the success of Redux were: the problem and the constraints.

Redux was developed to solve a problem. The problem was explained by Dan Abramov one year ealier when he introduced Redux. It was not yet another library. It was a library that solved a problem. Time Traveling and Hot Reloading were the stress test for Redux.

The second key take away, the contrainss, were another key factor to its success. Redux managed to shield away the problem with a simple API and a thoughtful way to solve the problem of state management itself.

[You can watch the talk](https://www.youtube.com/watch?v=uvAXVMwHJXU) too. However, maybe it makes sense to watch it after the book introduced you the basics of Redux.

- TODO introduction: http://blog.isquaredsoftware.com/presentations/2017-02-react-redux-intro/#/58 or check if it deals with react-redux

- http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-1/

## Basics in Redux

On the [official Redux website](http://redux.js.org/) it says: *"Redux is a predictable state container for JavaScript apps."*. It can be used standalone or in connection with with libraries, like React and Angular, to manage state in JavaScript applications.

Redux adopted a handful of constraints from the Flux architecture but not all of them. It has Actions that encapsulate information about the state update. It has a Store to save the state too. However, the Store is a singleton. Thus there are no multiple Stores like there used to be in the Flux architecture. In addition, there is no single Dispatcher. Instead, Redux uses multiple Reducers. Basically Reducers pick up the information from Actions and "reduce" it to a new state that is saved in the Store. When state in the Store is changed, the View can act on this.

{title="Code Playground",lang="javascript"}
~~~~~~~~
View -> Action -> Reducer(s) -> Store -> View
~~~~~~~~

Now the 100 million dollar question: Why is it called Redux? Because it combines the two words Reducer and Flux.

The abstract picture should be imagineable now. The state doesn't live in the View anymore, it is only connected the View. What does connected mean? It is connected on two ends, because it is part of the unidirectional data flow. One end is responsible to trigger an Action to update the state, the second end is resonspible to receive the state from the Store. The View can update according to the state and trigger state changes.

The View, in this case, would be React, but Redux could be used with any other library or standalone. After all, it is only a state management container.

### Action

An action in Redux is a JavaScript object. It has a type and a named payload. The type is often reffered as **action type**. While the type is a string literal, the payload can be everything. It can range from string or number to a complex object. Imagine the following action that is used in an application that manages Todos:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'ADD_TODO',
  todo: { id: '0', name: 'learn redux', completed: false },
}
~~~~~~~~

Executing an action is called **dispatch** in Redux. You can dispatch an action to alter the state in the Redux store. You only dispatch when you want to change the state. The dispatch of an action can be triggered in your View layer. It could be as simple as a click on a button.

In addition, the payload in a Redux action is not mandatory. You can define actions that have only an action type. That subject will be revisited later in the book.

Once an action is dispatched, it will come by all reducers in Redux.

### Reducer

A reducer is the next part in the chain. The View dispatches an action and the action object, with action type and payload, will pass through all reducers.

What's a reducer? A reducer is a pure function. It takes an input and always produces the same output when the input stays the same. It has no side-effects.

A reducer has two inputs: state and action. The state is the whole state object from the Redux store. The action is the dispatched action with a type and a payload. The reducer reduces the previous state and incoming action to a new state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
(state, action) => newState
~~~~~~~~

Apart from the functional programming principle that a reducer is a pure function without side-effects, it also embraces immutable data structures. It always returns a `newState` object without modifying the incoming `state` object. Thus the following, where the state is a list of todos, is not allowed:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function(state, action) {
  return state.push(action.todo);
}
~~~~~~~~

It would mutate the previous state instead of returning a new state object. The following is allowed because it keeps the previous state intact.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  return Object.assign({}, state, action.todo);
}
~~~~~~~~

By using the spread operator, the state, thus the list of todos, is spreaded into a new array. In addition, the newly added todo from the action is appended. The data structure stays immutable.

But what about the action type? Right now only the payload is used to produce a new state.

When an action object arrives at the reducers, the action type should be evaulated. Only when a reducer cares about the action type, it will produce a new state. Otherwise it simply returns the previous state. In JavaScript a switch case can help to branch into different action types. Redux makes use of it.

Imagine your Todo app would have a second action that toggles a Todo to completed or incomplete.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'TOGGLE_TODO',
  todoId: '0',
}
~~~~~~~~

The reducer would have to act on two actions now: `ADD_TODO` and 'TOGGLE_TODO'. By using a switch case statement, it would look like the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  switch(action.type) {
    case 'ADD_TODO' : {
      return Object.assign({}, state, action.todo);
    }
    case 'TOGGLE_TODO' : {
      // map over all todos
      // change only the matching todo to the toggled completed state
      // return remaining todos without changes
      const todos = state.map(todo =>
        todo.id === action.todo.id
          ? Object.assign({}, todo, { completed: !todo.completed })
          : todo
      );

      return Object.assign({}, todos, action.todo);
    }
    default : return state;
  }
}
~~~~~~~~

The JavaScript map functionality always returns a new array. It doesn't mutate the previous state and thus the state stays immutable.

Notice that Redux so far only uses plain JavaScript. There is no hidden magic. In order to keep it tidy, most often the different switch case branches get extracted as pure functions.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  switch(action.type) {
    case 'ADD_TODO' : {
      return applyAddTodo(state, action);
    }
    case 'TOGGLE_TODO' : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}

function applyAddTodo(state, action) {
  return Object.assign({}, state, action.todo);
}

function applyToggleTodo(state, action) {
  // map over all todos
  // change only the matching todo to the toggled completed state
  // return remaining todos without changes
  const todos = state.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );

  return Object.assign({}, todos, action.todo);
}
~~~~~~~~

## Store

- reducers are registered in a store
- dispatch
- listen

What about the inital state in my application?

- store initial
- reducer initial

# Hands On: Redux Standalone

- little hands on implemention

## Fine Grained Actions

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

## Fine Grained Reducers

- more to know

### Combined Reducer

- so far you have learned that there are multiple reducers, but all the previous examples used only one reducer (TODO check if this is still true)
-the reducer can care about multiple actions though, but how to use multiple reducers?

- combine helper

### Nested Reducers

- TODO first check if that is not already done by extracting functuons
- Reducers can be nested
- my personal opinon: it is good to know that these cna be nested, but I didn't see is as widely adopted in React and Redux applications. it can make the rreasoning about the state updates more diffictul again

### Action to Reducer like 1:N

- that will be explained in greater detail in the Command and Event patterns in Redux

## Hands On: Redux Standalone Advanced

- combined reducer
- action creators

- nested reducer ?
- action more abstracted ?

## Hands On: Redux from Scratch

- read up several articles that already made this
- https://news.ycombinator.com/item?id=14273549
- https://zapier.com/engineering/how-to-build-redux/

## Redux in React

- react-redux
- connect HOC that listen to the store
- container (or connected components) as gateways to state
- State -> View

## Gateway Components

- mix up presenter and container

## How to achieve Immutability?

- so far Object.assign
- ES6!

- other libraties immutableJs, moba?
- https://medium.com/@fastphrase/should-i-use-immutablejs-with-redux-58f88d6fd81e
- two ways: beginner and large scaling applications can use helper libs
- my recommendation is to stay to ES6 only
- problems like: you would run into state hydrations: rehydrating and dehydrating

## Selectors

- plain selectors
- reselect with memoize

## Normalized State

- normalizr
- https://www.reddit.com/r/reactjs/comments/5tnk9e/react_redux_how_to_use_normalized_state/

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

# Redux Forms

- naive usage 1 chapter without library
- library 2 chapter
- under the hood 3 chapter, you can do your own HOC libraries that connect to the Redux store!


