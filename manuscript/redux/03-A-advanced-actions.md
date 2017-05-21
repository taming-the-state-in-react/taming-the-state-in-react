## Advanced Actions

You have learned about actions in a previous chapter. However, there are more fine grained details that I want to cover in this chapter. The same applies for reducers that will be covered in the following chapter. Therefore it would be a requirement that you feel confidtent with the learnings from the previous chapter. All the following learnings are not mandatory to write applications in Redux, but they teach best practices in Redux. In an advanced application you would want to know about these.

### Minimum Action Payload

Do you recall the action from a previous chapter that added a todo? It was something like the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'ADD_TODO',
  todo: { id: '0', name: 'learn redux', completed: false },
}
~~~~~~~~

As you can see, the `completed` property is defined as false. In addition you have seen that the action and reducer from the previous chapter did work under these circumstances. However, a rule of thumb in Redux is to keep the action payload to a minimum.

In the example, when you want to add a todo in a todo application, it would need at least the unique identifier and a name of a todo. But the `completed` property is unneccesary. The assumption is that every todo that is added to the store will be incomplete. It wouldn't make sense in a puristic todo application to add completed todos. Therefore not the action would take care about the property but the reducer.

Instead of simply passing the whole todo object into the list of todos:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
  return state.concat(action.todo);
}
~~~~~~~~

You can add the `completed` property:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
  const todo = Object.assign({}, action.todo, { completed: false });
  return state.concat(todo);
}
~~~~~~~~

And omit it in the action:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'ADD_TODO',
  todo: { id: '0', name: 'learn redux' },
}
~~~~~~~~

Now you defined only the necessary information in the action. Nevertheless, if the todo application decides at some point to add incompleted todos in the first place, you can add it in the action again and leave it out in the reducer. However, ultimatively keeping the payload in the action to a minimum is a best practice in Redux.

### Action Type

Actions get evaluated in reducers by action type. The action type is the glue between both parts even though actions and reducers can be defined independently. To make the application more robust, you should extract the action type as a variable. Otherwise you can run into typos where an action never reaches a reducer.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'ADD_TODO';
# leanpub-end-insert

const action = {
# leanpub-start-insert
  type: ADD_TODO,
# leanpub-end-insert
  todo: { id: '0', name: 'learn redux' },
};

const toggleTodoAction = {
# leanpub-start-insert
  type: TOGGLE_TODO,
# leanpub-end-insert
  todo: { id: '0' },
};

function reducer(state, action) {
  switch(action.type) {
# leanpub-start-insert
    case ADD_TODO : {
# leanpub-end-insert
      return applyAddTodo(state, action);
    }
# leanpub-start-insert
    case TOGGLE_TODO : {
# leanpub-end-insert
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

There is another benefit in extracting the action type as variable. Because action, reducer and action type are loosely coupled, you could define them in separate files. You would only need to import the action type for to use specific actions and reducers.

### Action Creator

Action creators add another layer on top that often leads to confusion when learning Redux. Action creators are not mandatory, but they are convenient to use.

So far, you have dispatched an action as plain action object:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const ADD_TODO = 'ADD_TODO';

store.dispatch({
  type: ADD_TODO,
  todo: { id: '0', name: 'learn redux' },
});
~~~~~~~~

However, action creators encapsule the action with its action type and optional payload. In addition, they give you the flexibility to pass any payload. After all, they are only functions.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doAddTodo(id, name) {
  return {
    type: ADD_TODO,
    todo: { id, name },
  };
}
~~~~~~~~

Now, you can use them by invoking the function in your dispatch method:

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch(doAddTodo('0', 'learn redux'));
~~~~~~~~

Action creators return a plain action. Once again, it is not mandatory to use them, but it adds convenience.

### Optional Payload

In the book it was earlier mentioned that actions don't need to have a payload. Only the action type is required.

For instance, imagine you want to login into your todo application. Therefore you need to open up a modal where you can enter your credentials: email and password. You wouldn't need a payload for your action in order to open a modal. You only need to signalize by dispatching an action that the modal state should be open.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'OPEN_LOGIN_MODAL',
}
~~~~~~~~

A reducer would take care of it and set the state of a `isLoginModalOpen` property to true. While it is good to know that the payload is not mandatory in actions, the last example can lead to bad practice. Because you already know that you would need a second action to close the modal again.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'CLOSE_LOGIN_MODAL',
}
~~~~~~~~

A reducer would set the `isLoginModalOpen` property in the state to false. That's verbose, because you already need two actions to alter only one property in the state.

By planning your actions thoughtfully, you avoid these bad practices and keep your actions on a higher level of abstraction. If you would use the optional payload for the action, you could solve login scenario in only one action instead of two actions. The `isLoginModalOpen` property would be dynamically set in the action rather than hardcoded in a Reducer.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'TOGGLE_LOGIN_MODAL',
  isLoginModalOpen: true,
}
~~~~~~~~

By using an action creator, the payload is up to the usage of the action creator.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doToggleLoginModal(open) {
  type: 'TOGGLE_LOGIN_MODAL',
  isLoginModalOpen: open,
}
~~~~~~~~

In idiomatic Redux, actions should always try to stay on an abstract level rather than on a concrete level. Otherwise, you will end up with duplications and verbose actions. However, don't worry too much about it for now. This will be explained in more detail in another chapter. (TODO check again when other chapter is written Command Evtn pattern.)

### Payload Structure

Again you will encounter a best practice that is not mandatory in Redux. So far, the payload was dumped without much thought in the actions. Now imagine an action that has a larger payload than a simple todo.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'ADD_ASSIGNED_TODO',
  todo: { id: '0', name: 'learn redux' },
  assignedTo: { id: '99' name: 'Robin' },
}
~~~~~~~~

The properties would add up, but mask the one most important property: the type. Therefore you should treat action type and payload on the same level, but nest the payload one level deeper.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'ADD_ASSIGNED_TODO',
  payload: {
    todo: { id: '0', name: 'learn redux' },
    assignedTo: { id: '99' name: 'Robin' },
  },
}
~~~~~~~~

The refactoring ensures that type and payload are visible on first glance.

## Hands On: Redux Standalone with advanced Actions

Let's dip again into the Redux Playground with the acquired knowledge about actions. You can take again the [JS Bin project that you have done in the last chapter](https://jsbin.com/kopohur/7/edit?html,js,console). The project will be used to show the advanced actions. You can try it on your own. Otherwise the following part will guide you through the refactorings.

The minimum action payload is a quick refactoring. You can omit the `completed` in the action and add it to the reducer functionality.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
# leanpub-start-insert
  const todo = Object.assign({}, action.todo, { completed: false });
  return state.concat(todo);
# leanpub-end-insert
}

...

store.dispatch({
  type: 'ADD_TODO',
# leanpub-start-insert
  todo: { id: '0', name: 'learn redux' },
# leanpub-end-insert
});

store.dispatch({
  type: 'ADD_TODO',
# leanpub-start-insert
  todo: { id: '1', name: 'learn mobx' },
# leanpub-end-insert
});
~~~~~~~~

The next step is the extraction of the action type from the actions and reducer. It should be defined as a variable and can be replaced in the reducer.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'TOGGLE_TODO';

function reducer(state, action) {
  switch(action.type) {
# leanpub-start-insert
    case ADD_TODO : {
# leanpub-end-insert
      return applyAddTodo(state, action);
    }
# leanpub-start-insert
    case TOGGLE_TODO : {
# leanpub-end-insert
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

In addition, you can use the variable in the dispatched actions.

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch({
# leanpub-start-insert
  type: ADD_TODO,
# leanpub-end-insert
  todo: { id: '0', name: 'learn redux' },
});

store.dispatch({
# leanpub-start-insert
  type: 'ADD_TODO',
# leanpub-end-insert
  todo: { id: '1', name: 'learn mobx' },
});

store.dispatch({
# leanpub-start-insert
  type: TOGGLE_TODO,
# leanpub-end-insert
  todo: { id: '0' },
});
~~~~~~~~

In the next step you can introduce action creators to your Todo application.

First, you can define them:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doAddTodo(id, name) {
  return {
    type: ADD_TODO,
    todo: { id, name },
  };
}

function doToggleTodo(id) {
  return {
    type: TOGGLE_TODO,
    todo: { id },
  };
}
~~~~~~~~

Second, you can use them in your dispatch invocations:

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch(doAddTodo('0', 'learn redux'));
store.dispatch(doAddTodo('1', 'learn mobx'));
store.dispatch(doToggleTodo('0'));
~~~~~~~~

There were two more advanced topics about actions in this chapter: optional payload and payload structure. The first topic wouldn't apply in the current application. Every action has to have a payload. The second topic could be applied. However, the payload is small and thus doesn't need to be deeply restructured with a payload property.

The [final Todo application can be found in this JS Bin](https://jsbin.com/kopohur/8/edit?html,js,console). You can do further experiments with it before you continue with the next chapter.




