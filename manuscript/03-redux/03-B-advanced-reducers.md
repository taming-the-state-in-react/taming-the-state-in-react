## Advanced Reducers

Apart from the advanced topics about actions, there is more to know about reducers, too. Again, not everything is mandatory, but you should at least know about the following things to embrace best practices, common usage patterns and practices on scaling your state architecture.

### Initial State

So far, you have provided your store with an initial state. It was an empty list of todos.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(reducer, []);
~~~~~~~~

That's the initial state for the whole Redux store. However, you can apply the initial state on a more fine-grained level. Before you dispatch your first action, the Redux store will initialize by running through all reducers once. You can try it by removing all of your dispatches in the editor and add a `console.log()` in the reducer. You will see that it runs with an initializing action once, even though there is no dispatched action.

The initializing action, that is received in the reducer, is accompanied by the initial state that is specified in the `createStore()` function. However, if you leave out the initial state in the store initialization, the incoming state in the reducer will be `undefined`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(reducer);
~~~~~~~~

That's where you can opt-in to specify an initial state on a more fine-grained level. If the incoming state is `undefined`, you can default with a [JavaScript ES6 default parameter](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Default_parameters) to a default initial state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function reducer(state = [], action) {
# leanpub-end-insert
  switch(action.type) {
    case TODO_ADD : {
      return applyAddTodo(state, action);
    }
    case TODO_TOGGLE : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

The initial state will be the same as before, but you defined it on a more fine-grained level. Later on, that will help you when you specify more than one reducer and your state object becomes more complex because it is split up into multiple reducers. Then every reducer can define its fine-grained initial state.

### Nested Data Structures

The initial state is an empty list of todos. However, in a growing application you want to operate on more than todos. You might have a `currentUser` object that represents the logged in user in your application. In addition, you want to have a `filter` property to filter todos by their `completed` property. In a growing application, more objects and arrays will gather in the global state. That's why your initial state shouldn't be an empty array but an object that represents the state object. This object then has nested properties for `todos`, `currentUser` and `filter`.

Disregarding the initial state in the reducer from the last chapter, you would define your initial state in the store like in the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const initialState = {
  todos: [],
};
const store = createStore(reducer, initialState);
~~~~~~~~

Now you can use the space horizontally in your `initialState` object. It might grow to the following at some point:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const initialState = {
  currentUser: null,
  todos: [],
  filter: 'SHOW_ALL',
};
~~~~~~~~

If you get back to your Todo application, you will have to adjust your reducer. The reducer deals with a list of todos as state, but now it is a complex global state object.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
  switch(action.type) {
    case TODO_ADD : {
      return applyAddTodo(state, action);
    }
    case TODO_TOGGLE : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}

function applyAddTodo(state, action) {
# leanpub-start-insert
  const todo = Object.assign({}, action.todo, { completed: false });
  const todos = state.todos.concat(todo);
  return Object.assign({}, state, { todos });
# leanpub-end-insert
}

function applyToggleTodo(state, action) {
# leanpub-start-insert
  const todos = state.todos.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );
  return Object.assign({}, state, { todos });
# leanpub-end-insert
}
~~~~~~~~

Those **nested data structures** are fine in Redux, but you want to avoid **deeply nested data structures**. As you can see, it adds complexity to create your new state object. Another chapter which you will read up later in the book will pick up this topic again. It will showcase how you can avoid deeply nested data structures by using a neat helper library with the name normalizr.

### Combined Reducer

The next chapter is crucial to understand the principles of scaling state by using substates in Redux. In the previous chapters, you have heard about multiple reducers, but haven't used them yet. As you have learned, a reducer grows horizontally when it deals with more than one action type. But this doesn't scale at some point. You want to split your reducer up into two reducers or introduce another reducer right from the beginning. Imagine you had a reducer that served your todos, but you want to have a reducer for the `filter` state for your todos eventually. Another use case could be to have a reducer for the `currentUser` that is logged in into your Todo application.

You can already see a pattern on how to separate your reducers. It is usually by domain like todos, filter, or user. A todo reducer might be responsible to add, remove, edit and complete todos. A filter reducer is responsible to manage the filter state of your todos. A user reducer cares about user entities that could be the `currentUser` who is logged in in your application or a list of users who are assigned to todos. That's where you could again split up the user reducer to a `currentUser` reducer and a `assignedUsers` reducer. You can imagine how this approach, introducing reducers by domain, scales very well.

Let's enter **combined reducers** to enable you using multiple reducers. Redux gives you a helper to combine multiple reducers into one root reducer: `combineReducers()`. The function takes an object as argument which can have multiple reducers assigned to their state which they are going to manage.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
});
~~~~~~~~

Afterward, the `rootReducer` can be used to initialize the Redux store instead of the single todo reducer. After all, only one reducer is needed to initialize the Redux store. So it has to be a combination of multiple reducers when having more than one reducer.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(rootReducer);
~~~~~~~~

That's it for the initialization of the Redux store with combined reducers. But what about the reducers themselves? There is already one reducer that cares about the todos, you would only have the rename it.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function todoReducer(state, action) {
# leanpub-end-insert
  switch(action.type) {
    case TODO_ADD : {
      return applyAddTodo(state, action);
    }
    case TODO_TOGGLE : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

The `filterReducer` is not there yet. It would look something like the following to set a filter for the todos (e.g. SHOW_ALL, SHOW_COMPLETED, SHOW_INCOMPLETED):

{title="Code Playground",lang="javascript"}
~~~~~~~~
function filterReducer(state, action) {
  switch(action.type) {
    case FILTER_SET : {
      return applySetFilter(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

Now, there comes the important part: The `combineReducers()` function introduces an intermediate state layer for the global state object. The global state object, when using the combined reducers as shown before, would look like the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  todoState: ...,
  filterState: ...,
}
~~~~~~~~

The property keys for the intermediate layer are those defined in the `combineReducers()` function. However, in your reducers the incoming state is not the global state object anymore. It is their defined substate from the `combinedReducers()` function. The `todoReducer` doesn't know anything about the `filterState` and the `filterReducer` doesn't know anything about the `todoState`.

The `filterReducer` and `todoReducer` can use the JavaScript ES6 default parameter to define their initial state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function todoReducer(state = [], action) {
# leanpub-end-insert
  switch(action.type) {
    case TODO_ADD : {
      return applyAddTodo(state, action);
    }
    case TODO_TOGGLE : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}

# leanpub-start-insert
function filterReducer(state = 'SHOW_ALL', action) {
# leanpub-end-insert
  switch(action.type) {
    case FILTER_SET : {
      return applySetFilter(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

Now your `apply` function in the reducers have to operate on these substates again.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
# leanpub-start-insert
  const todo = Object.assign({}, action.todo, { completed: false });
  return state.concat(todo);
# leanpub-end-insert
}

function applyToggleTodo(state, action) {
# leanpub-start-insert
  return state.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );
# leanpub-end-insert
}

# leanpub-start-insert
function applySetFilter(state, action) {
  return action.filter;
}
# leanpub-end-insert
~~~~~~~~

It seems like a lot of hassle to define multiple reducers and combining them eventually. But it is crucial knowledge to split up your global state into substates. These substates can be managed by their reducers who only operate on these substates. In addition, each reducer can be responsible to define the initial substate.

### Clarification for Initial State

The last chapters moved around the initial state initialization from `createStore()` to the reducer(s) a few times. You might wonder where to initialize your state after all. Therefore, you have to distinguish whether you are using combined reducers or only one plain reducer.

**One plain reducer:** When using only one plain reducer, the initial state in `createStore()` dominates the initial state in the reducer. The initial state in the reducer only works when the incoming initial state is `undefined` because then it can apply the default state from the default parameter. But the initial state is already defined in `createStore()` and thus utilized by the reducer.

**Combined reducers:** When using combined reducers, you can embrace a more nuanced usage of the state initialization. The initial state object that is used for the `createStore()` function doesn't have to include all substates that are introduced by the `combineReducers()` function. Thus, when a substate is `undefined`, the reducer can define the default substate. Otherwise, the default substate from the `createStore()` is used.

### Nested Reducers

By now, you know two things about scaling reducers in a growing application that demand sophisticated state management:

* a reducer can care about different action types
* a reducer can be split up into multiple reducers yet be combined as one root reducer for the store initialization

These steps are used to scale the reducers horizontally (even though combined reducers add at least one vertical level). A reducer operates on the global state or on a substate when using combined reducers. However, you can use nested reducers to introduce vertically levels of substate too.

Take for example the `todoReducer` that operates on a list of todos. From a technical perspective, a list of todos has single todo entities. So why not introduce a nested reducer that deals with the todo substate as entities?

{title="Code Playground",lang="javascript"}
~~~~~~~~
function todoReducer(state = [], action) {
  switch(action.type) {
    case TODO_ADD : {
# leanpub-start-insert
      return state.concat(todoEntityReducer(undefined, action));
# leanpub-end-insert
    }
    case TODO_TOGGLE : {
# leanpub-start-insert
      return state.map(todo => todoEntityReducer(todo, action));
# leanpub-end-insert
    }
    default : return state;
  }
}

# leanpub-start-insert
function todoEntityReducer(state, action) {
  switch(action.type) {
    case TODO_ADD : {
      return applyAddTodo(state, action);
    }
    case TODO_TOGGLE : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}

function applyAddTodo(state, action) {
  return Object.assign({}, action.todo, { completed: false });
}

function applyToggleTodo(todo, action) {
  return todo.id === action.todo.id
    ? Object.assign({}, todo, { completed: !todo.completed })
    : todo
}
# leanpub-end-insert
~~~~~~~~

You can use nested reducers to introduce intermediate layers for substates. In addition, they can be reused. You might run into cases where you can reuse a nested reducer somewhere else. However, while nested reducers can give you a better picture on your state, they add more levels of complexity to your state, too. You should follow the practice of not nesting your state too deeply in the first place. Then you won't run into nested reducers often.

### Hands On: Redux Standalone with advanced Reducers

Let's dip into the Redux Playground again with the acquired knowledge about reducers. Again, you can take the [JS Bin project that you have done in the last chapter](https://jsbin.com/kopohur/29/edit?html,js,console). The project will be used to show the advanced reducers. You can try it on your own. Otherwise, the following part will guide you through the refactorings. First, let's add the second reducer to filter the todos.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const FILTER_SET = 'FILTER_SET';

function filterReducer(state = 'SHOW_ALL', action) {
  switch(action.type) {
    case FILTER_SET : {
      return applySetFilter(state, action);
    }
    default : return state;
  }
}

function applySetFilter(state, action) {
  return action.filter;
}
~~~~~~~~

The reducer has an initial state. There is no action yet that serves the reducer though. You can add a simple action creator that only receives a string as argument for the filter property. You will experience later on, how this can be used in a real application.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doSetFilter(filter) {
  return {
    type: FILTER_SET,
    filter,
  };
}
~~~~~~~~

Second, you can rename your first reducer to `todoReducer` and give it an initial state of an empty list of todos.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function todoReducer(state = [], action) {
# leanpub-end-insert
  switch(action.type) {
    case TODO_ADD : {
      return applyAddTodo(state, action);
    }
    case TODO_TOGGLE : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

The initial state isn't initialized in the `createStore()` function anymore. It is initialized on a more fine-grained level in the reducers. When you recap the last lessons learned from the advanced reducers chapter, you will notice that you spared the back and forth with the initial state. Now, the `todoReducer` still operates on the `todos` substate and the new `filterReducer` operates on the `filter` substate. As third and last step, you have to combine both reducers to get this intermediate layer of substates in the first place. In the JS Bin you have Redux available as global variable to get the `combineReducer` function. Otherwise, you could import it with JavaScript ES6.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const rootReducer = Redux.combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
});
~~~~~~~~

Now you can use the combined `rootReducer` in the store initialization.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(rootReducer);
~~~~~~~~

When you run your application again, everything should work. You can now use the `filterReducer` as well.

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch(doAddTodo('0', 'learn redux'));
store.dispatch(doAddTodo('1', 'learn mobx'));
store.dispatch(doToggleTodo('0'));
# leanpub-start-insert
store.dispatch(doSetFilter('COMPLETED'));
# leanpub-end-insert
~~~~~~~~

The Todo application favors initial state in reducers over initial state in `createStore()`. In addition, it will not use a nested `todoEntityReducer` for the sake of keeping the reducer hierarchy simple. Even though there is one intermediate state layer because of the combined reducers now.

The [final Todo application can be found in this JS Bin](https://jsbin.com/kopohur/30/edit?html,js,console). You can do further experiments with it before continuing with the next chapter.
