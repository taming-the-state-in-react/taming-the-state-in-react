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

## Selectors

- plain selectors
- reselect with memoize

- denormaliez state, or by id

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
