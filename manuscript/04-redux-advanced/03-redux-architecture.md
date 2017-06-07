## Redux State as Architecture

The book taught you the practical usage of Redux. You have learned about the main parts in the Redux state management architecture: actions, reducers and the store.

{title="Code Playground",lang="javascript"}
~~~~~~~~
Action -> Reducer(s) -> Store
~~~~~~~~

The chain is connected to the view by something (e.g. react-redux with `mapStateToProps()` and `mapDispatchToProps()`) that enables you to write connected components. These components have access to the Redux store. They are used to receive state or to alter the state. They are a specialized case of a container component in the presenter and container pattern when using components.

{title="Code Playground",lang="javascript"}
~~~~~~~~
View -> (mapDispatchToProps) -> Action -> Reducer(s) -> Store -> (mapStateToProps) -> View
~~~~~~~~

All other components are not aware of any local or sophisticated state management solution. They only receive props, except they have their own local state management (such as `this.state` and `this.setState()` in React).

State can be received directly by operating on the state object or indirectly by selecting it with selectors.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// directly from state object
state.something;

// indirectly from state object via selector
const getSomething = (state) => state.something;
~~~~~~~~

State can be altered by dispatching an action directly or by using action creators that return an action object.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// dispatching an action directly
dispatch({ type: 'ANY_TYPE', payload: anyPayload });

// dispatching an action indirectly via action creator
function doAnything(payload) {
  return {
    type: 'ANY_TYPE',
    payload,
  };
}
dispatch(doAnything(anyPayload));
~~~~~~~~

In order to keep your state predictable and manageable in the reducers, you can apply techniques for an improved state structure. You can normalize your state to have always a single source of truth. That means you don't have to operate on duplicated entities, but only on one reference of the entity. In addition, it keeps the state flat. It is easier to manage only by using spread operators.

Around these practical usages, you have learned several supporting techniques. There are tons of opinionated ways to organize your folders and files. The book showcased two of the main approaches, but they vary in their execution from developer to developer, team to team or company to company. Nevertheless, you should always bear in mind to keep Redux at a top level. It is not used to manage the state of one single component. Instead it is used to wire dedicated components to the store in order to enable them to alter and to retrieve the state from it.

Coupling actions and reducers is fine, but always think twice when adding another action type. For instance, perhaps a action type could be reused in another reducer. When reusing action types, you avoid to end up with fat thunks when using Redux thunk. Instead of dispatching several actions, your thunk could dispatch only one abstract action that is reused in more than one reducer.

You have learned that you can plan your state management ahead. There are use cases where local state makes more sense than sophisticated state. Both can be used and should be used in a scaling application. By combining local state to the native local storage of the browser, you can give the user of your application an improved UX. In addition, you can plan the state ahead too. Think about view state and entitiy state and where it should live in your application. You can give your reducers differenct domains as their ownership such as todoReducer, filterReducer and notificationReducer. However, once you have planned your state management and state, don't stick to it. When slaing your application, always revisit those things to apply refactorings. That will help you to keep your state manageable, maintainable and predictable in the long run.

## Hands On: Hacker News with Redux