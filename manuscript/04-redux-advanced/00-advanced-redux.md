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

The TodoItem component itself would stay the same. It still gets the `todo` item and the `onToggleTodo` handler as arguments. In addition, you can see two more concepts that were explained earlier. First, the selector grows in complexity because it gets optional arguments to select derived properties fromt the state. Second, the `mapStateToProps()` function makes use of the incoming props from the `TodoList` component that uses the `ConnectedTodoItem` component.

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

### Hands On: Todo with Selectors

- derived properties (visibility filter todos in React Redux example)

## Asynchronous Actions

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

# Challenge: Snake with Redux

- show off command (only one reducer cares, but refactor it to local state) vs event (multiple reducers care) pattern

# Hands On: Hacker News with Redux

- you have built one in plain React in the Road to learn React
- show off normalizr