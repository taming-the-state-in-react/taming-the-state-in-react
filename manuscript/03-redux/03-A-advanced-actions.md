## Advanced Actions

You have learned about actions in a previous chapter. However, there are more fine-grained details that I want to cover in this chapter. The same applies for reducers. Both concepts will be covered in the following chapters in more detail. Therefore, it would be a requirement that you feel confident with the learnings from the previous chapter. Not all of the following learnings are mandatory to write applications in Redux, but they teach best practices and common usage patterns. In a larger sized application, you would want to know about these topics.

### Minimum Action Payload

Do you recall the action from a previous chapter that added a todo item? It was something like the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'TODO_ADD',
  todo: { id: '0', name: 'learn redux', completed: false },
}
~~~~~~~~

As you can see, the `completed` property is defined as false boolean. In addition, you saw that the action and reducer from the previous chapter did work under these circumstances. However, a rule of thumb in Redux is to keep the action payload to a minimum.

In the example, when you want to add a todo in a Todo application, it would need at least the unique identifier and a name of a todo. But the `completed` property is unnecessary. The assumption is that every todo that is added to the store will be incomplete in the beginning. It wouldn't make sense in a Todo application to add completed todos, would it? Therefore, not the action would take care of the `completed` property but the reducer.

Instead of simply passing the whole todo object from the action into the list of todos in your reducer function:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
  return state.concat(action.todo);
}
~~~~~~~~

You can add the `completed` property as hardcoded property to your reducer function:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
  const todo = Object.assign({}, action.todo, { completed: false });
  return state.concat(todo);
}
~~~~~~~~

Finally, you can omit it in the action's payload:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'TODO_ADD',
  todo: { id: '0', name: 'learn redux' },
}
~~~~~~~~

Now, you only defined the necessary payload in the action to add a todo item to the list in the store. Nevertheless, if at some point the Todo application decides to add uncompleted todos in the first place, you can add it in the action again and leave it out of the reducer. Ultimately, keeping the payload in the action to a minimum should be a best practice in Redux.

### Action Type

Actions get evaluated in reducers by action type. The action type is the glue between both parts even though actions and reducers can be defined independently. To make the application more robust, you should extract the action type as a variable. Otherwise, you can run into typos where an action never reaches a reducer because you misspelled it.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const TODO_ADD = 'TODO_ADD';
const TODO_TOGGLE = 'TODO_TOGGLE';
# leanpub-end-insert

const action = {
# leanpub-start-insert
  type: TODO_ADD,
# leanpub-end-insert
  todo: { id: '0', name: 'learn redux' },
};

const toggleTodoAction = {
# leanpub-start-insert
  type: TODO_TOGGLE,
# leanpub-end-insert
  todo: { id: '0' },
};

function reducer(state, action) {
  switch(action.type) {
# leanpub-start-insert
    case TODO_ADD : {
# leanpub-end-insert
      return applyAddTodo(state, action);
    }
# leanpub-start-insert
    case TODO_TOGGLE : {
# leanpub-end-insert
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

There is another benefit in extracting the action type as variable. Because action, reducer and action type are loosely coupled, you can define them in separate files. You would only need to import the action type to use them only in specific actions and reducers. After all, action types could be used in multiple reducers too. This use case will be covered in another chapter when it comes to the advanced reducers.

### Action Creator

Action creators add another layer on top, which often leads to confusion when learning Redux. Action creators are not mandatory, but they are convenient to use. So far, you have dispatched an action as plain action object:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const TODO_ADD = 'TODO_ADD';

store.dispatch({
  type: TODO_ADD,
  todo: { id: '0', name: 'learn redux' },
});
~~~~~~~~

Action creators encapsulate the action with its action type and optional payload in a reusable function. By having a function for your action, you gain the flexibility to pass any payload to the action as arguments. After all, they are only pure functions which return an object.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doAddTodo(id, name) {
  return {
    type: TODO_ADD,
    todo: { id, name },
  };
}
~~~~~~~~

Now, you can use the action creator by invoking it as function in your dispatch method:

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch(doAddTodo('0', 'learn redux'));
~~~~~~~~

Action creators return a plain action (object). Once again, it is not mandatory to use them, but it adds convenience and makes your code more readable in the long run. In addition, you can test action creators independently as functions. Last but not least, these action creators stay reusable because they are functions.

### Optional Payload

In the book, it was mentioned earlier that actions don't need to have a payload. Only the action type is required. For instance, imagine you want to login into your Todo application. Therefore, you need to open up a modal where you can enter your credentials: email and password. You wouldn't need a payload for your action in order to open a modal. You only need to signalize that the modal state should be stored as "open" by dispatching an action.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'LOGIN_MODAL_OPEN',
}
~~~~~~~~

A reducer would take care of it and set the state of a `isLoginModalOpen` boolean to true. While it is good to know that the payload is not mandatory in actions, the last example can lead to a bad practice. It is because you may already know that you would need a second action to close the modal again:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'LOGIN_MODAL_CLOSE',
}
~~~~~~~~

Again a reducer would set the `isLoginModalOpen` boolean in the state to false. That's verbose, because you already need two actions to alter only one property in the state. By planning your actions thoughtfully, you avoid these bad practices and keep your actions on a higher level of abstraction. If you used the optional payload for the action, you could solve login scenario in only one action instead of two actions. The `isLoginModalOpen` property would be dynamically passed in the action rather than being hardcoded in a reducer.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'LOGIN_MODAL_TOGGLE',
  isLoginModalOpen: true,
}
~~~~~~~~

By using an action creator, the payload can be passed in as arguments and thus stays flexible. Afterward, only one action instead of two is needed.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doToggleLoginModal(open) {
  return {
    type: 'LOGIN_MODAL_TOGGLE',
    isLoginModalOpen: open,
  };
}
~~~~~~~~

In Redux, actions should always try to stay on an abstract level rather than on a concrete level. Otherwise, you will end up with duplications and verbose actions. However, don't worry too much about it for now. This will be explained in more detail in another chapter in this book that is about commands and events.

### Payload Structure

Again you will encounter a best practice that is not mandatory in Redux. So far, the payload was dumped without much thought in the actions. Now imagine an action that has a larger payload than a simple todo. For instance, the action payload should clarify to whom the todo is assigned.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'TODO_ADD_ASSIGNED',
  todo: { id: '0', name: 'learn redux' },
  assignedTo: { id: '99' name: 'Robin' },
}
~~~~~~~~

The properties would add up horizontally in the object, but mask the one most important property, which is the action type, when they become too many. Therefore, you should treat action type and payload on the same level, but nest the payload itself one level deeper as the two abstract properties.

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  type: 'TODO_ADD_ASSIGNED',
  payload: {
    todo: { id: '0', name: 'learn redux' },
    assignedTo: { id: '99' name: 'Robin' },
  },
}
~~~~~~~~

The refactoring ensures that type and payload are visible on first glance. As said, it is not mandatory to do so and often adds more complexity. But in larger applications it keeps your action creators more readable.

### Hands On: Redux Standalone with advanced Actions

Let's dip into the Redux Playground again with the acquired knowledge about actions. You can take the [JS Bin project that you have done in the last chapter](https://jsbin.com/kopohur/28/edit?html,js,console) again. The project will be used to show the advanced actions. You can try it on your own. Otherwise, the following part will guide you through the refactorings.

The minimum action payload is a quick refactoring. You can omit the `completed` property in the action and add it to the reducer functionality.

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
  type: 'TODO_ADD',
# leanpub-start-insert
  todo: { id: '0', name: 'learn redux' },
# leanpub-end-insert
});

store.dispatch({
  type: 'TODO_ADD',
# leanpub-start-insert
  todo: { id: '1', name: 'learn mobx' },
# leanpub-end-insert
});
~~~~~~~~

The next step is the extraction of the action types from the actions and reducer. They should be defined as variables and thus can be replaced in the reducer.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const TODO_ADD = 'TODO_ADD';
const TODO_TOGGLE = 'TODO_TOGGLE';

function reducer(state, action) {
  switch(action.type) {
# leanpub-start-insert
    case TODO_ADD : {
# leanpub-end-insert
      return applyAddTodo(state, action);
    }
# leanpub-start-insert
    case TODO_TOGGLE : {
# leanpub-end-insert
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

In addition, you can use the variables in the dispatched actions too.

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch({
# leanpub-start-insert
  type: TODO_ADD,
# leanpub-end-insert
  todo: { id: '0', name: 'learn redux' },
});

store.dispatch({
# leanpub-start-insert
  type: 'TODO_ADD',
# leanpub-end-insert
  todo: { id: '1', name: 'learn mobx' },
});

store.dispatch({
# leanpub-start-insert
  type: TODO_TOGGLE,
# leanpub-end-insert
  todo: { id: '0' },
});
~~~~~~~~

In the next step, you can introduce action creators to your Todo application. First, define them as functions:

{title="Code Playground",lang="javascript"}
~~~~~~~~
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
~~~~~~~~

And second, use them in your dispatch functions to return the action objects:

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch(doAddTodo('0', 'learn redux'));
store.dispatch(doAddTodo('1', 'learn mobx'));
store.dispatch(doToggleTodo('0'));
~~~~~~~~

There were two more advanced topics about actions in this chapter: optional payload and payload structure. The first topic wouldn't apply in the current application. Every action has to have a payload in this scenario. The second topic could be applied. However, the payload is too small at this point in time and thus doesn't need to be restructured with an intermediate payload property.

The [final Todo application can be found in this JS Bin](https://jsbin.com/kopohur/29/edit?html,js,console). You can do further experiments with it before you continue with the next chapter.
