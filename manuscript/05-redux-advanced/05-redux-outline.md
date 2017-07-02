# Redux Ecosystem Outline

- state keys, gateway components ,  ... (LINK them)
- line mark erikson repository

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

## Redux Devtools

- http://blog.jakoblind.no/2017/06/15/how-to-become-a-more-productive-react-developer/

## Challenge: Hacker News with beyond Redux

 - extended with router (dismissed), typed, redux form (search field), using es6, folder organization
