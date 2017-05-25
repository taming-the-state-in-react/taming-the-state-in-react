## Redux in React

In the last chapters you got to know plain Redux. It helps you to manage a predictable state container. However, you want to use this state in an application eventually. It can be any JavaScript application that has to deal with state management. Even though Redux is a JavaScript library, the general paradigm could be applied in any programming language.

The state management in single page applications (SPAs) is one of these use cases where Redux can be applied. These applications are usually built with a SPA framework (Angular) or view layer library (React, Vue). The book focuses on React, but you can apply the following learning to other solutions too.

The following scenarios will omit the practice of local state management and apply sophisticated state management with Redux. Even though these examples could live without Redux, because they wouldn't run into scaling state management issues.

### Connecting State

On the one hand you have React. ...

On the other hand you have Redux. You know how to manage your state in Redux now. It has basically two steps. First, you initialize everything by setting up reducer(s) and optional action creators. The (combined) reducer is used to create the Redux store. Second, you can interact with the store by dispatching actions, subscribing the store and getting the current state from the store.

These three interactions from the second step need to be accessed from your view layer. As mentioned, the view layer can be anything, but to keep it focused it is React in this book. How can `dispatch`, `subscribe` and `getState` be accessed?

- react-redux

### Connect as Higher Order Component

- ref article

### Managing State from Everywhere

- containers can be placed everywhere
- container (or connected components) as gateways to state

### Showing State from Everywhere

- State -> View
- presenters that only get callbacks to alter state
- don't know that they are connected to local state or Redux store
- showcase with presenter

### Hands On: Todo application with React and Redux

- JS Bin: react with todo app

### Hands On: Snake with React and Redux

- create-react-app