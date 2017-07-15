# Redux Ecosystem Outline

After learning the basics and advanced techniques in Redux and applying them on your own in an application, you are ready to explore the Redux ecosystem. The Redux ecosystem is huge and cannot be covered in one book. However this chapter attempts to outline different paths you can take to explore the world of Redux. Apart from outlining these different paths, a couple of topics will be revisisted as well to give you a richer toolset when using Redux.

Before you are left alone with the least chapters covering Redux, I want to make you aware of [this repository](https://github.com/markerikson/redux-ecosystem-links) by Mark Erikson. It is a categorized list of Redux related addons, libraries and articles. If you get stuck at some point, want to find a solution for your problem or are just curious about the ecosystem, check out the repository.

## Redux DevTools

The Redux DevTools are essential for many developers when implementing Redux applications. It improves the Redux development workflow by offering a rich set of features such as inspecting the state and action payload, time traveling and realtime optimizations.

How does it work? Basically you have two choices to use the Redux DevTools. Either you can integrate it directly into your project by using its node package with npm or you can install the official browser extension. While the former comes with an implementation setup in your application, the former can be simply installed for your browser without changing your implementation.

The most obvious feature is to inspect actions and state. Rather than using the redux-logger, you can use the Redux DevTools to get insights into these information. You can follow each state change by inspecting the action and the state.

Another great feature is the possibility to time travel. In Redux you dispatch actions and travel from one state to another state. The Redux DevTools enable you to time travel. You can move back in time by reverting actions. For instance, that way you wouldn't need to reload your browser anymore to follow a set of actions to get to a specific application state. You could simply alter the actions in between by using the Redux DevTools. You can trace back what action led to which state.

In addition, you can persist your Redux state when doing page reloads with Redux DevTools. That way, you don't need to perform all the necessary actions to get to a specific state anymore. You simply reload the page and keep the same application state. This enables you to debug your application when having one specific application state.

However, there are more neat features that you might enjoy while developing a Redux application. You can find all information about the Redux DevTools in the [official repository](https://github.com/gaearon/redux-devtools).

## Revisited: connect

- its not only redux, but react-redux
https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options

## handleActions, createActions

- TODO

## Redux Forms

- naive usage 1 chapter without library
- library 2 chapter
- under the hood 3 chapter, you can do your own HOC libraries that connect to the Redux store!

## Routing with Redux

- how to
- clear state?
- react-router everyone is using
- http://redux.js.org/docs/advanced/UsageWithReactRouter.html

- https://www.reddit.com/r/reactjs/comments/6d82li/is_it_advisable_to_integrate_reactrouter_with/

- https://medium.com/handlebar-labs/use-redux-to-manage-react-navigation-state-b6d639497143

## Typed Redux

- flow
- alternative: typescript

## Fullstack Redux

- sevrer side rendering, store is not a singleton there! thats why it is good to never pass around the store directly, you dont do it in connected components and do not do it in asynchronours action creators.
- sockets

- state keys, gateway components ,  ... (LINK them)

## Challenge: Hacker News with beyond Redux

 - extended with router (dismissed), typed, redux form (search field), using es6, folder organization
