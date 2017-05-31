# Redux, but Advanced

You have learned about Redux standalone and Redux in React. You already would be able to build applications with it. Before you dive deeper into Redux, I recommend you to experiment with your recent learnings and apply them on smaller applications.

The following chapter guides you through more advanced topics in Redux. You will get to know the middleware in Redux and get answers on how to keep your state immutable. It will show you to normalize your state and how to retrieve derived state by using selectors. Last but not least, you will learn about asynchronous actions. These actions will enable you to make asynchronous requests to retrieve data from an external REST API and save it to your global state object.

## Middleware in Redux

In Redux you can use a middleware. Every dispatched action in Redux flows through this middleware. You can opt-in a specif behavior in between dispatching an action and the moment it reaches the reducer.

There are useful libraries out there to opt-in behaviors into your Redux middleware. In the following, you will get to know one of them: [redux-logger](https://github.com/evgenyrodionov/redux-logger). When you use it, it doesn't change anything in your application in production. But it will make your life easier as developer when dealing with Redux in development. What does it do? It simply logs the actions in your developer console with `console.log()`. As a developer, it gives you clarity on which action is dispatched and how the previous and the new state are structured.

But where to apply the middleware? The stores can be initilaized with a middleware. The `createStore()` functionality from Redux takes as third argument an enhancer. Redux comes with one of these enhancers: `applyMiddleware()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { applyMiddleware, createStore } from 'redux';

const store = createStore(
  reducer,
  undefined,
  applyMiddleware(...)
);
~~~~~~~~

Now, when using redux-logger, you can pass a `logger` instance to the `applyMiddleware()` function.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { applyMiddleware, createStore } from 'redux';
# leanpub-start-insert
import { createLogger } from 'redux-logger'

const logger = createLogger();
# leanpub-end-insert

const store = createStore(
  reducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(logger)
# leanpub-end-insert
);
~~~~~~~~

That's it. Now every action should be visible in your developer console when dispatching them. You don't have to take care about this anymore. Your state changes become more predictable as developer.

The `applyMiddleware()` functionality takes any number of middleware: `applyMiddleware(firstMiddleware, secondMiddleware, ...);`. The action will flow through all middleware before it reaches the reducers. Sometimes you have to make sure to apply them in the correct order. For instance, the `redux-logger` middleware must be last in the middleware chain in order to output the correct actions and states.

Nevertheless, that's only the redux-logger middleware. On your way to implement Redux applications, you will surely find out more about useful features that can be used with a middleware. Most of theses features are already taken care of in libraries. For instance, asynchronous actions in Redux are possible by using the Redux middleware. These will be described later in this book.

## Immutable State

Redux embraces an immutable state. Your reducers will always return a new state object. You will never mutate the incoming state. Therefore you might have to get used to different JavaScript functionalities and syntax to embrace immutable data structures.

So far, you have used built-in JavaScript functionalities to keep your data structures immutable. Such as

* `array.map()`
* `array.concat(item)`

for arrays or `Object.assign()` for objects. All of these functionalities return new instances of arrays or objects. Often you have to read the official JavaScript documentation to make sure that they return a new instance. Otherwise you would violate the constraints of Redux.

But it doesn't end here. You should know about your tools to keep data structures immutable. There are a handful of third-party libraries that can support you in keeping them immutable.

* [immutable.js](https://github.com/facebook/immutable-js)
* [mori.js](https://github.com/swannodette/mori)
* [seamless-immutable.js](https://github.com/rtfeldman/seamless-immutable)
* [baobab.js](https://github.com/Yomguithereal/baobab)

But they come with three drawbacks. First, they add another layer of complexity to your application. Second, you have to learn yet another library. And third, you have to hydrate your data, because most of these libraries wrap your vanilla JavaScript objects and arrays into a library specific immutable data object and array. It is an immutable data object/array in your Redux store, but once you want to use it in React you have to transform it into a plain JavaScript object/array.

Personally I would recommend to use such libraries only in two scenarios:

* You are not comfortable to keep your data structures immutable with JavaScript ES5, JavaScript ES6 and beyond.
* You want to improve the performance of immutable data structures when using huge amounts of data.

If both statements are false, I would advice you to stick to plain JavaScript. As you have seen the built-in JavaScript functionalities already help a lot. In JavaScript ES6 and beyond you get one more functionality to keep your data structures immutable: [spread operators](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator). Spreading an object or array into a new object or new array always gives you a new object or new array.

Do you recall your reducer that handles the todos from your Todo application?

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
  const todo = Object.assign({}, action.todo, { completed: false });
  return state.concat(todo);
}

function applyToggleTodo(state, action) {
  return state.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );
}
~~~~~~~~

You can express it with JavaScript ES6 too.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
# leanpub-start-insert
  const todo = { ...action.todo, completed: false };
  return [ ...state, todo ];
# leanpub-end-insert
}

function applyToggleTodo(state, action) {
  return state.map(todo =>
    todo.id === action.todo.id
# leanpub-start-insert
      ? { ...todo, completed: !todo.completed }
      : todo
# leanpub-end-insert
  );
}
~~~~~~~~

JavaScript gives you enough tools to keep your data structures immutable. THere is no need to use third-party library except for the two mentioned use cases. However, there might be a third use case where such library would help: deeply nested data structures in Redux that need to be kept immutable. It is true that it becomes more diffictult to keep data structures immutable when they are deeply nested. however, as mentioned ealier in the book, it is bad practice to have deeply nested data structures in Redux in the first place. That's were the next chapter of the book comes into play.

## Normalized State

A best practice in Redux is a flat state. You don't want to maintain a immutable state when it is deeply nested. It becomes even with spread operators tedious and unreadable. But often you have no control over your data structure, because it comes from a backend application via its API. When having a deeply nested data structure, you have two options:

* saving it in the store as it is and postpone the problem
* saving it as normalize data structure in the store

You should prefer the second option. You only deal once with the problem and all subsequent parts in your application will be grateful for it. Let's run through one scenario to illustrate the normalization of data. Imagine you have a deeply nested data structure:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const data = {
  todos: [
    {
      id: '0',
      name: 'create redux',
      completed: true,
      assignedTo: {
        id: '99',
        name: 'Dan Abramov',
      },
    },
    {
      id: '1',
      name: 'create mobx',
      completed: true,
      assignedTo: {
        id: '77',
        name: 'Michel Weststrate',
      },
    }
  ],
};
~~~~~~~~

Both library creators, Dan Abramov and Michel Weststrate, did their homework: they created popular libraries for state management. The first option, as mentioned, would be to save the todos as they are in the store. The todos itself would have the deeply nested information of the assigned user. Now, when an action would want to correct the assigned user, the reducer would have to deal with the deeply nested data structure.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const ASSIGNED_TO_CHANGE = 'ASSIGNED_TO_CHANGE';

store.dispatch({
  type: ASSIGNED_TO_CHANGE,
  payload: {
    todoId: '0',
    name: 'Dan Abramov and Andrew Clark',
  },
});

function todosReducers(state = [], action) {
  switch(action.type) {
    case ASSIGNED_TO_CHANGE : {
      return applyChangeAssignedTo(state, action);
    }

    ...

    default : return state;
  }
}

function applyChangeAssignedTo(state, action) {
  return state.map(todo => {
    if (todo.id === action.payload.todoId) {
      const assignedTo = { ...todo.assignedTo, name: action.payload.name };
      return { ...todo, assignedTo };
    } else {
      return todo;
    }
  });
}
~~~~~~~~

The further you have to reach into a deeply nested data structure, the more you have to be careful to keep your data structure immutable. Each level of nested data adds more tedious work of maintaing it. Therefore, you should use a library called [normalizr](https://github.com/paularmstrong/normalizr). The library uses schema defintions to transform deeply nested data structures into dictionaries that have entities and a corresponending list of ids.

How would that look like? Let's take the previous todos as example. First, you would have to define once schemas for your entities:

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { schema } from 'normalizr';

const assignedToSchema = new schema.Entity('assignedTo');

const todoSchema = new schema.Entity('todo', {
  assignedTo: assignedToSchema,
});
~~~~~~~~

Second, you can normalize your data whenever you want:

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { schema, normalize } from 'normalizr';
# leanpub-end-insert

const assignedToSchema = new schema.Entity('assignedTo');

const todoSchema = new schema.Entity('todo', {
  assignedTo: assignedToSchema,
});

# leanpub-start-insert
const normalizedData = normalize(data.todos, [ todoSchema ]);
# leanpub-end-insert
~~~~~~~~

The output would be the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  entities: {
    assignedTo: {
      77: {
        id: "77",
        name: "Michel Weststrate"
      },
      99: {
        id: "99",
        name: "Dan Abramov"
      }
    },
    todo: {
      0: {
        assignedTo: "99",
        completed: true,
        id: "0",
        name: "create redux"
      },
      1: {
        assignedTo: "77",
        completed: true,
        id: "1",
        name: "create mobx"
      }
    }
  },
  result: ["0", "1"]
}
~~~~~~~~

The deeply nested data became a flat data structure grouped by entities. Each entity can reference another entity by its id. Now you could have one reducer that stores and deals with the `assignedTo` entities and one reducer that deals with the `todo` entities. The data structure is flat and grouped by entities and thus easier to access and to manipulate.

There is another benefit in normalizing your data. When your data is denormalized, entities are often duplicated in your nested data structure. Imagine the following object with todos.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const data = {
  todos: [
    {
      id: '0',
      name: 'write book',
      completed: true,
      assignedTo: {
        id: '55',
        name: 'Robin Wieruch',
      },
    },
    {
      id: '1',
      name: 'call it taming the state',
      completed: true,
      assignedTo: {
        id: '55',
        name: 'Robin Wieruch',
      },
    }
  ],
};
~~~~~~~~

If you would store such denormalized data in your Redux store, you will likely run into an issue. What's the problem with the denormalized data structure? Imagine you want to update the name 'Robin Wieruch' of all `assignedTo` properties. You would have to run through all todos in order to update all `assignedTo` properties with the id '55'. The problem: there is no single source of truth. You will likely forget to update an entity and have stale state eventually. THerefore the best practice is to store your state normalized so that each entitiy can be a single source of truth. There will be no duplication of entities and thus no stale state when updating the single source of truth. Each todo will reference to the updated `assignedTo` entitity:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  entities: {
    assignedTo: {
      55: {
        id: "55",
        name: "Robin Wieruch"
      }
    },
    todo: {
      0: {
        assignedTo: "55",
        completed: true,
        id: "0",
        name: "write a book"
      },
      1: {
        assignedTo: "55",
        completed: true,
        id: "1",
        name: "call it taming the state"
      }
    }
  },
  result: ["0", "1"]
}
~~~~~~~~

In conclusion, normalizing your state has two benefits. It keeps your state flat and thus easier manageable with immutable data structures. In addition, it groups entities to single sources of truth without any duplications. When you normalize your state, you will automatically get groupings of entities that could lead to their own reducers managing them.

There exists yet another benefit when normalizing your state. It is about denormalization: how to component retrieve the normalized state? You will learn it in the next chapter.

## Selectors

In advanced Redux there is another concept to know about it. While normalizing state is about how to store your state, selecting state is about how to retrieve your state. This part in Redux is called selectors. They are pure functions that return derived properties from your state. It can be that they only return a substate of your global state or that they already preprocess your state to return derived properties.

### Plain Selectors

Selectors usually follow a similar syntax. The mandatory argument of a selector is the state from where it has to select from. There can be other argument that are in a supportive role for the selector to select or derive the correct properties.

{title="Code Playground",lang="javascript"}
~~~~~~~~
(state) => derived properties
~~~~~~~~

Selectors are not mandatory. When thinking about all the parts in Redux, only the action(s), the reducer(s) and the Redux store are a binding requirement. Similar to action creators, selectors can be used to achieve an improved developer experience in a Redux architecture. How does a selector look like? It is a plain function, as mentioned, that could live anywhere. However, you would use it, when using Redux in React, in your `mapStateToProps()` function.

Instead of retrieving the state explicitly:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function mapStateToProps(state) {
  return {
    todos: state.todos,
  };
}
~~~~~~~~

You would retrieve it implicit via a selector:

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function getTodos(state) {
  return state.todos;
}
# leanpub-end-insert

function mapStateToProps(state) {
  return {
# leanpub-start-insert
    todos: getTodos(state),
# leanpub-end-insert
  };
}
~~~~~~~~

It is similar to the action and reducer concept. Instead of manipulating the state directly in the Redux store, you will use action(s) and reducer(s) to alter it indirectly. The same applies for reducers that don't retrieve the derived properties directly but indirectly from the global state.

Why is that an advantage? There are several benefits. A selector can be reused. You will run more often into cases where you select the derived properties. That's always a good sign to use a function in the first place. In addition, selectors can be tested separately. They are pure functions and thus an easily testable part in the architecture. Last but not least, deriving properties from state can become a complex undertaking in a scaling application. As mentioned, a selector could get optional arguments to derive more sophistitated properties from the state. The selector function itself would become more complex, but it would be encapsulated in one function rather than, for instance in React and Redux, in a `mapStateToProps()` function.

### Denormalize State in Selectors

In the last chapter, about normalizing your state, there was one benefit left unexplained. It is about selecting normalized state. Personally I would argue a normalized state structure makes it much more convenient to select state from it. When we recall the normalized state structure, it looked something like the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  entities: ...
  ids: ...
}
~~~~~~~~

It would look similar to this when you would apply it to the todos in the Todo application:

{title="Code Playground",lang="javascript"}
~~~~~~~~
// state
[
  { id: '0', name: 'learn redux' },
  { id: '1', name: 'learn mobx' },
]

// normalized state
{
  entities: {
    0: {
      id: '0',
      name: 'learn redux',
    },
    1: {
      id: '1',
      name: 'learn redux',
    },
  },
  ids: ['0', '1'],
}
~~~~~~~~

If you recall the Redux in React chapter, there you passed the todos from the `TodoApp` component down to the whole component tree. How would you solve this with the normalized state from above?

You can try to solve the question on your own. Open up the Todo application in your Redux Playground and use the normalized state from above as initial state. You would have to adapt the reducer to handle the normalized state structure though. If you don't come up with a solution, which is fine, you can keep reading to get to know about the solution.

Assuming that the reducer would store the state in a normalized immutable data structure, you would only pass the list of todd ids to your `TodoList` component. Because this component manages the list and not the entities itself, it makes perfect sense that it only gets the lsit with references to the entitirs.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function TodoList({ todosAsIds }) {
  return (
    <div>
      {todosAsIds.map(todoId => <ConnectedTodoItem
        key={todoId}
        todoId={todoId}
      />)}
    </div>
  );
}

function getTodosAsIds(state) {
  return state.todo.ids;
}

function mapStateToProps(state) {
  return {
    todosAsIds: getTodosAsIds(state),
  };
}

const ConnectedTodoList = connect(mapStateToProps)(TodoList);
~~~~~~~~

Now the ConnectedTodoItem, that already passes the `onToggleTodo` handler via the `mapDispatchToProps()` function to its plain `TodoItem` component, would retrieve the todo entity matching to the incoming `todoId` property.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function getTodoAsEntity(state, id) {
  return state.todo.entities[id];
}

function mapStateToProps(state, props) {
  return {
    todo: getTodoAsEntity(state, props.todoId),
  };
}

function mapDispatchToProps(dispatch) {
  return {
     onToggleTodo: id => dispatch(doToggleTodo(id)),
  };
}

const ConnectedTodoItem = connect(mapStateToProps, mapDispatchToProps)(TodoItem);
~~~~~~~~

The `TodoItem` component itself would stay the same. It still gets the `todo` item and the `onToggleTodo` handler as arguments. In addition, you can see two more concepts that were explained earlier. First, the selector grows in complexity because it gets optional arguments to select derived properties fromt the state. Second, the `mapStateToProps()` function makes use of the incoming props from the `TodoList` component that uses the `ConnectedTodoItem` component.

As you can see, the normlaized state requires to use more connected component. More components are responsible to select their needed derived properties. But in a growing application, following this pattern can make it easier to reason about it. You only pass properties that are really necessary to the component. In the last case, the `TodoList` component only cares about a list of references and the `TodoItem` component itself cares about the entitiy that is selected by using the reference passed down by the `TodoList` component.

There exists another way, when using normalizr, to denormalize your normalized state. The previous scenario allowed you to pass only the minimum of properties to the components. Each componetn was responsible to select its state. In this scenario, you will denormalize your state in one component while the other components don't need to care about it.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { denormalize, schema } from 'normalizr';

const todoSchema = new schema.Entity('todos');
const todosSchema = { todos: [ todoSchema ] };

function TodoList({ todos }) {
  return (
    <div>
      {todos.map(todo => <ConnectedTodoItem
        key={todo.id}
        todo={todo}
      />)}
    </div>
  );
}

function getTodos(state) {
  const entities = state.todo.entities;
  const ids = state.todo.ids;
  return denormalize(ids, [ todoSchema ], entities);
}

function mapStateToProps(state) {
  return {
    todos: getTodos(state),
  };
}

const ConnectedTodoList = connect(mapStateToProps)(TodoList);
~~~~~~~~

In this scenario, the whole normalized data structure gets normalized in the selector. You will have the whole list of todos in your `TodoList` component. The `TodoItem` component wouldn't need to take care about the denormlization.

### Reselect

- reselect with memoize

### Hands On: Todo with Normalized Data and Selectors

In the Todo application, you could refactor everything to use normalized data and selectors now. Let's open up again the Redux Playground. You can use the [JS Bin project from the Redux in React chapter](https://jsbin.com/kopohur/23/edit?html,js,console,output). It imports all necessary libraries.

In the beginning, you would have to exchange the initial state to an already normalized initial state. The initial state would go in the `todosReducer` and not in the `createStore()` initialization.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const initialTodoState = {
  entities: {
    0: {
      id: '0',
      name: 'learn redux',
    },
    1: {
      id: '1',
      name: 'learn redux',
    },
  },
  ids: ['0', '1'],
};

function todosReducer(state = initialTodoState, action) {
# leanpub-end-insert
  switch(action.type) {
    case ADD_TODO : {
      return applyAddTodo(state, action);
    }
    case TOGGLE_TODO : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

Now you have to adapt the `apply` functions to handle your new data structure.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
  const todo = Object.assign({}, action.todo, { completed: false });
  const entities = Object.assign({}, state.entities, todo);
  const ids = state.ids.concat(action.todo.id);
  return Object.assign({}, state, ids, entities);
}

function applyToggleTodo(state, action) {
  const todo = state.entities[action.todo.id];
  const toggledTodo = Object.assign({}, todo, { completed: !todo.completed });
  const entities = Object.assign({}, state.entities, toggledTodo);
  return Object.assign({}, state, entities);
}
~~~~~~~~

That's it. The last step is to adjust your `mapPropsToState()` functions that they can return the derived properties from the state. Afterwards you can use these in your components. First, let's do it for the `TodoList` component:

{title="Code Playground",lang="javascript"}
~~~~~~~~
~~~~~~~~

Second, you can do it for your `TodoItem` component:


{title="Code Playground",lang="javascript"}
~~~~~~~~
~~~~~~~~

Now try to refactor the Todo application on your own to use selectors instead of retrieving the properties direclty from the state. Otherwise you can use the following solution.

- derived properties (visibility filter todos in React Redux example)

# Challenge: Snake with Redux

- show off command (only one reducer cares, but refactor it to local state) vs event (multiple reducers care) pattern

## Asynchronous Actions

In the book, you have only used synchronous actions by now. There is no delay of the action dispatching involved. Yet sometimes you want to delay an action. The example can be a simple one: Imagine you want to have a notifaction for your application user when a todo item got created. The notification should hide after one second. The first action would show the notification and set a `isShowingNotification` property to true. If the `isShowing` status of the notification resides in the store, you would need a way to delay a second action to hide the notification again. In a simple scenario it would look like the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch({ type: 'NOTIFICATION_SHOW', text: 'Todo created.' });
setTimeout(() => {
  store.dispatch({ type: 'NOTIFICATION_HIDE' })
}, 1000);
~~~~~~~~

There is nothing against a simple `setTimeout()` function in your application. It can be used in the context of Redux and React. Sometimes it is easier to remember the basics in JavaScript than trying to apply yet another library to fix a problem. Since you know about actions creators, the next step could be to extract these to actions into according action creators and action types.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const NOTIFICATION_SHOW = 'NOTIFICATION_SHOW';
const NOTIFICATION_HIDE = 'NOTIFICATION_HIDE';

function doShowNotification(text) {
  return {
    type: NOTIFICATION_SHOW,
    text,
  };
}

function doHideNotification() {
  return {
    type: NOTIFICATION_HIDE,
  };
}
# leanpub-end-insert

# leanpub-start-insert
store.dispatch(doShowNotification('Todo created.'));
# leanpub-end-insert
setTimeout(() => {
# leanpub-start-insert
  store.dispatch(doHideNotification());
# leanpub-end-insert
}, 1000);
~~~~~~~~

There are two problems in a growing application now. First, you would have to duplicate the logic of the delayed action every time. Second, once your application dispatches multiple notifications at various places at the same time, the first running action that hides the notifications would hide all of them. The solution would be to give each notification an identifier. Both problems can be solved by extracting the functionality into a function.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// actions
const NOTIFICATION_SHOW = 'NOTIFICATION_SHOW';
const NOTIFICATION_HIDE = 'NOTIFICATION_HIDE';

function doShowNotification(text, id) {
  return {
    type: NOTIFICATION_SHOW,
    text,
    id,
  };
}

function doHideNotification(id) {
  return {
    type: NOTIFICATION_HIDE,
    id,
  };
}

// extracted functionality
let naiveId = 0;
function showNotificationWithDelay(dispatch, text) {
  dispatch(doShowNotification(text, naiveId));
  setTimeout(() => {
    dispatch(doHideNotification(naiveId));
  }, 1000);

  naiveId++;
}

// usage
showNotificationWithDelay(store.dispatch, 'Todo created.');
~~~~~~~~

The extracted function gets control over the `dispatch()` method from the Redux store, because it needs to dispatch a delayed action.

**Why not passing the Redux store instead?** Usually you want to avoid to pass the store around directly. You have encountered the same reasoning in the book when weaving the Redux store for the first time into your React application. You want to make the functionalities of the store available, but not the entire store itself. That's why you only have the `dispatch()` method and not the entire store in your `mapDispatchToProps()` function when using react-redux. A connected component has never access to the store directly and thus no other functionalities should have direct access to it. You will learn more about this best practice in a chapter that is about server side rendering (TODO check if I write it).

The pattern from above suffices for simple Redux applications that need a delayed action. However, in scaling applications it has a drawback. The approach creates two types of action creators. While there are synchronours action creators that can be dispacthed directly, there are pseudo asynchronours action creators too. These pseudo asynchronours action creators cannot be dispacthed directly but have to accept the dispatch method as argument. Wouldn't it be great to use both types of actions the same without worrying to pass around the dispatch method and any asynchronours or synchronours behavior?

### Redux Thunk

- TODO https://www.reddit.com/r/reactjs/comments/6e0vgt/is_my_understanding_of_reduxthunk_correct/?st=1Z141Z3&sh=255adc41
- TODO https://medium.com/@talkol/redux-thunks-dispatching-other-thunks-discussion-and-best-practices-dd6c2b695ecf

The previous question led Dan Abramov, the creator of Redux, think about a general pattern to the problem of asynchronours actions. He came up with the library called [redux-thunk](https://github.com/gaearon/redux-thunk) to legitimize the concept. Synchronours and asynchronours action creators should be dispatched in a similar way from a Redux store. It is used as middleware in your Redux store.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { createStore, applyMiddleware } from 'redux';
# leanpub-start-insert
import thunk from 'redux-thunk';
# leanpub-end-insert

...

const store = createStore(
  rootReducer,
# leanpub-start-insert
  applyMiddleware(thunk)
# leanpub-end-insert
);
~~~~~~~~

Basically it creates a middleware for your actions creators. In this middleware you are enabled to dispatch asynchronours actions. Apart from dispatching objects, you can dispatch functions with Redux Thunk. You have always dispatched objects before in this book. An action itself is an object and an action creator returns an action object.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// with plain action
store.dispatch({ type: 'NOTIFICATION_SHOW', text: 'Todo created.' });

// with action creator
store.dispatch(doShowNotification('Todo created.'));
~~~~~~~~

However, now you can pass the dispatch method a function too. The function will always give you access to the dispatch method again.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// with thunk function
let naiveId = 0;
store.dispatch(function (dispatch) {
  dispatch(doShowNotification('Todo created.', naiveId));
  setTimeout(() => {
    dispatch(doHideNotification(naiveId));
  }, 1000);

  naiveId++;
});
~~~~~~~~

The dispatch method of the Redux store when using Redux Thunk will differentiate between passed objects and functions. The passed function is called a **thunk**. You can dispatch as many actions synchronoursly and asynchronously as you want in a thunk. When a thunk is growing and handles complex business logic at some point in your application, it is called a **fat thunk**. You can extract a thunk function as an asyncrhonours action creator, that is a higher order fucntion and returns the thunk function, and can be dispatched the same way as an synchronours action creator.

{title="Code Playground",lang="javascript"}
~~~~~~~~
let naiveId = 0;
function showNotificationWithDelay(text) {
  return function (dispatch) {
    dispatch(doShowNotification(text, naiveId));
    setTimeout(() => {
      dispatch(doHideNotification(naiveId));
    }, 1000);

    naiveId++;
  };
}

store.dispatch(showNotificationWithDelay('Todo created.'));
~~~~~~~~

It is similar to the solution without Redux Thunk. However, this time you don't have to pass around the dispatch method and instead have access to it in the returned thunk function. Now, when using it in a React component, the component still only executes a callback function that it receives via its props. The connected component then dispatches a action, regardless of the action being synchroinsly or asynchrounsly, in the `mapDispatchToProps()` function.

That are the basics of Redux Thunk. There are a few more things that are good to know about it:

* **getState():** A thunk function gives you as second argument the `getState()` method of the Redux store: `function (dispatch, getState)`. However, you should generally avoid it. It's best practice to pass all necessary state to the action creator instead of retrieving it in the thunk.
* **Promises:** Thunks work great in combination with promises. You can return a promise from your thunk and use it, for instance, to wait for its completion: `store.dispatch(showNotificationWithDelay('Todo created.')).then(...)`.
* **Recursive Thunks:** The dispatch method in a thunk can again be used to dispatch an asynchronours action. Thus you can apply the thunk pattern recursively.

- TODO https://decembersoft.com/posts/what-is-the-right-way-to-do-asynchronous-operations-in-redux/

### Redux Saga

The whole concept around asynchronours actions led to a handful of libraries that solve this issue. Redux Thunk was only the first one introduced by Dan Abramov. However, he agrees that there are use cases where Redux Thunk doesn't solve the problem. Redux Thunk should be used when there are asynchronours actions. But when there are more complex scenarios around it, there are advanced solutions for asynchronours actions. This chapter shows you one of these solutions, Redux Saga, which is one of the most popular asynchrnours actions libraries for Redux.

- dealing with side-effects, so far not spoken about
- like side threats that only deal with side-effects
- you can control these with redux actions and dispatch new actions in them
- has access to full state too, similiar to Redux Thunk

- builds up on JavaScript ES6 Generators
- more about generators: https://redux-saga.js.org/docs/ExternalResources.html
- code looks synchronous, similar to async await, but with a few more need features that can be used in Redux Saga

- mature applications

### More Alternatives

- obsrvable: comparison to Saga http://stackoverflow.com/questions/40021344/why-use-redux-observable-over-redux-saga
- redux cycle: valid alternative for reactive programming

## Hands On: Todo with Notifications

- redux thunk

## Challenge: Snake with Redux

- extract the timeout functionality into a simple delayed action first, but then use redux thunk and/or redux saga to accomplish it
