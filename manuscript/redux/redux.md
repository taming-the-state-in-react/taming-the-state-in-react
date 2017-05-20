# Redux

Redux is one of the libraries that helps you implementing sophistaicated state management in your applications. It is one of the solutions you would take in a scaling application in order to tame the state. A React application is a perfect fit for Redux, yet other libraries and frameworks highly adopted it as well.

**Why is Redux that popular in the JavaScript community?** In order to answer that question, I have to go a bit into the past of JavaScript applications. In the beginning, there was one library to rule them all: jQuery. It was mainly used to manipulate the DOM, to amaze with animations and to implement reusbale widgets. It was the number one library in JavaScript. There was no way around it. However, the usage of jQuery skyrocketed and applications grew in size. They grew in JavaScript size. Eventually the code in those applications became a mess, because there was no proper architecture around it. The infamous spaghetti code came surfaced in JavaScript applications. It was about time for noveau solutions to emerge which should be beyond jQuery. These libraries, most of them frameworks, would bring the tools for architectures and opinionated approaches. They were called single page applications (SPAs).

Single page applications became popular when the first generation of frameworks and libraries, among them Angular 1, Ember and Backbone, were released. Suddenly developers had frameworks to build scaling frontend applications. However, with every new technology there will be new problems. Every framework had a different approach for state management. For instance, Angular 1 used the famous two-way data binding. It embraced a bidirectional data flow. Only after applications scaled, the problem in state management became widely known.

During that time React was released by Facebook. It was the second generation of SPA solutions. Compared to the first generation SPA solutions, it was only the view layer. It came with an own state management solution though.

In React the principle of the unidirectional data flow became popular. State management should be more predictable in order to reason about it. Yet, the local state management wasn't sufficient at some point. React applications scaled very well, but ran into problems of predictable and maintainable state management too. That was the time when Facebook introduced the Flux architecture.

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

### Reducer

- functiona: pure functions,

### Actions

### Action Types

### Action Creator

## Store

- dispatch
- listen

# Hands On: Redux Standalone

- little hands on implemention

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

- ES6!

- other libraties immutableJs, moba?
- https://medium.com/@fastphrase/should-i-use-immutablejs-with-redux-58f88d6fd81e
- two ways: beginner and large scaling applications can use helper libs
- my recommendation is to stay to ES6 only
- problems like: you would run into state hydrations: rehydrating and dehydrating

## Nested Reducer

- TODO

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


