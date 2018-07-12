# Redux State Structure and Retrieval

In the previous chapters, you have learned about Redux standalone and Redux in React. You would already be able to build smaller and medium sized applications with it. Before you dive deeper into Redux, I recommend you to experiment with your recent learnings and apply them in your applications. If you jump straight into the next chapters, you may get the feeling that Redux is overkill (which it is for most of the applications out there). However, the coming chapters should give you advanced guidance when scaling your state management with Redux in larger applications. There are a couple of techniques you can apply then.

The following chapter guides you through more advanced topics in Redux to manage your state. You will get to know the middleware in Redux, you will learn more about a normalized and immutable state structure, and how to retrieve a substate in an improved way from the global state with selectors.

## Middleware in Redux

In Redux, you can use a middleware. Every dispatched action in Redux flows through this middleware. You can opt-in a specific feature in between of dispatching an action and the moment it reaches the reducer.

There are useful libraries out there to opt-in features into your Redux middleware. In the following, you will get to know one of them: [redux-logger](https://github.com/evgenyrodionov/redux-logger). When you use it, it doesn't change anything in your application. But it will make your life easier as developer when dealing with Redux. What does it do? It simply logs the actions in your browser's developer console with `console.log()`. As a developer, it gives you clarity on which action is dispatched and how the previous and the new state are structured.

But where to apply this middleware in Redux? It is the Redux store which can be initialized with it. The `createStore()` functionality from Redux takes as third argument a so called enhancer. The redux library comes with one of these enhancers: `applyMiddleware()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { applyMiddleware, createStore } from 'redux';

const store = createStore(
  reducer,
  undefined,
  applyMiddleware(...)
);
~~~~~~~~

If you don't have an initial state for your Redux state, you can use `undefined` for it. Now, when using redux-logger, you can pass a `logger` instance to the `applyMiddleware()` function.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { applyMiddleware, createStore } from 'redux';
# leanpub-start-insert
import { createLogger } from 'redux-logger';

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

That's it. Now every action should be visible in your browser's developer console when dispatching them. And thus your state changes become more predictable when developing without logging every action yourself.

The `applyMiddleware()` functionality takes any number of middleware: `applyMiddleware(firstMiddleware, secondMiddleware, ...);`. The action will flow through all middleware before it reaches the reducers. Sometimes, you have to make sure to apply them in the correct order. For instance, the `redux-logger` middleware must be last in the middleware chain in order to output the correct actions and states.

Nevertheless, that's only the redux-logger middleware. On your way to implement Redux applications, you will surely find out more about useful features that can be applied with the Redux middleware. Most of these features are already taken care of in libraries that you will find published with npm. For instance, asynchronous actions in Redux are possible by using the Redux middleware. These asynchronous actions will be explained later in this book.

## Immutable State

Redux embraces an immutable state. Your reducers will always return a new state object. You will never mutate the incoming state. Therefore, you might have to get used to different JavaScript functionalities and syntax to embrace immutable data structures.

So far, you have used built-in JavaScript functionalities to keep your data structures immutable. Such as `array.map()` and `array.concat(item)` for arrays or `Object.assign()` for objects. All of these functionalities return new instances of arrays or objects without altering the old arrays and objects. Often, you have to read the official JavaScript documentation to make sure that they return a new instance of the array or object. Otherwise, you would violate the constraints of Redux because you would mutate the previous instance which in this case is often the state or action.

But it doesn't end here. You should know about your tools to keep data structures immutable in JavaScript. There are a handful of third-party libraries that can support you in keeping them immutable.

* [immutable.js](https://github.com/facebook/immutable-js)
* [immer.js](https://github.com/mweststrate/immer)
* [mori.js](https://github.com/swannodette/mori)
* [seamless-immutable.js](https://github.com/rtfeldman/seamless-immutable)
* [baobab.js](https://github.com/Yomguithereal/baobab)

But all of them come with three drawbacks. First, they add another layer of complexity to your application. Second, you have to learn yet another library. And third, you have to dehydrate and rehydrate your data, because most of these libraries wrap your vanilla JavaScript objects and arrays into a library specific immutable data object and array. It is an immutable data object/array in your Redux store, but once you want to use it in React you have to transform it into a plain JavaScript object/array. Personally I would recommend to use such libraries only in two scenarios:

* You are not comfortable to keep your data structures immutable with JavaScript ES5, JavaScript ES6 and beyond.
* You want to improve the performance of immutable data structures when using huge amounts of data.

If both statements are false, I would highly recommend you to stick to plain JavaScript. As you have seen, the built-in JavaScript functionalities already help a lot for keeping data structures immutable. In JavaScript ES6 and beyond you get one more functionality to keep your data structures immutable: [spread operators](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator). Spreading an object or array into a new object or new array always gives you a new object or new array.

Do you recall how you added a new todo item or how you completed a todo item in your reducers?

{title="Code Playground",lang="javascript"}
~~~~~~~~
// adding todo
const todo = Object.assign({}, action.todo, { completed: false });
const newTodos = todos.concat(todo);

// toggling todo
const newTodos = todos.map(todo =>
  todo.id === action.todo.id
    ? Object.assign({}, todo, { completed: !todo.completed })
    : todo
  );
~~~~~~~~

If you added more JavaScript ES6 by using the spread operator, you can keep these even more concise.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// adding todo
const todo = { ...action.todo, completed: false };
const newTodos = [ ...todos, todo ];

// toggling todo
const newTodos = todos.map(todo =>
  todo.id === action.todo.id
    ? { ...todo, completed: !todo.completed }
    : todo
  );
~~~~~~~~

JavaScript gives you enough tools to keep your data structures immutable. There is no need to use a third-party library except for the two mentioned use cases. However, there might be a third use case where such library would help: deeply nested data structures in Redux that need to be kept immutable. It is true that it becomes more difficult to keep data structures immutable when they are nested. But, as mentioned earlier in the book, it is bad practice to have deeply nested data structures in Redux in the first place. That's where the next chapter of the book comes into play that can be used to keep your data structures flat in the Redux store.

## Normalized State

A best practice in Redux is a flat state structure. You don't want to maintain an immutable structure for your state when it is deeply nested. It becomes tedious and unreadable even with spread operators. But often you don't have control over your data structure, because it comes from a backend application by using its API. When having a deeply nested data structure, you have two options:

* saving it in the store as it is and postpone the problem (not good)
* saving it as normalized data structure in the store (good)

You should try to default to the second option. You only deal with the problem once and all subsequent parts in your application will be grateful for it. Let's run through one scenario to illustrate the normalization of data. Imagine you have the following nested data structure:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const todos = [
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
];
~~~~~~~~

Both library creators, Dan Abramov and Michel Weststrate, did a great job: they created popular libraries for state management. The first option, as mentioned, would be to save the todos as they are in the store. The todos themselves would have the deeply nested information of the `assignedTo` object within the `todo` object. Now, if an action wanted to correct an assigned user, the reducer would have to deal with the deeply nested data structure. Let's add Andrew Clark as creator of Redux.

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

function todoReducer(state = [], action) {
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

This would lead to the following list of todos in your Redux store after the action has been dispatched:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const todos = [
  {
    id: '0',
    name: 'create redux',
    completed: true,
    assignedTo: {
      id: '99',
      name: 'Dan Abramov and Andrew Clark',
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
];
~~~~~~~~

As you can see from the reducer, the further you have to reach into a deeply nested data structure, the more you have to be careful to keep your data structure immutable. Each level of nested data adds more tedious work of maintaining it. Therefore, you could use a library called [normalizr](https://github.com/paularmstrong/normalizr) to flatten (normalize) your state. The library uses schema definitions to transform deeply nested data structures into dictionaries that have entities and a corresponding list of ids.

What would that look like? Let's take the previous list of todo items as example. First, you would have to define schemas for your entities only once:

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
const normalizedData = normalize(todos, [ todoSchema ]);
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

The deeply nested data became a flat data structure grouped by entities. Each entity can reference another entity by its id. It is like you would keep these entities in a database. They are decoupled now. Afterward, in your Redux application, you could have one reducer that stores and deals with the `assignedTo` entities and one reducer that deals with the `todo` entities. The data structure is flat and grouped by entities and thus easier to access and to manipulate.

There is another benefit in normalizing your data. When your data is not normalized, entities are often duplicated in your nested data structure. Imagine the following object with todos.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const todos = [
  {
    id: '0',
    name: 'write a book',
    completed: true,
    assignedTo: {
      id: '55',
      name: 'Robin Wieruch',
    },
  },
  {
    id: '1',
    name: 'call it taming the state in react',
    completed: true,
    assignedTo: {
      id: '55',
      name: 'Robin Wieruch',
    },
  }
];
~~~~~~~~

If you store such denormalized data in your Redux store, you will likely run into an major issue: Imagine you want to update the name property `Robin Wieruch` of all `assignedTo` properties. You would have to run through all todos in order to update all `assignedTo` properties with the id `55`. The problem: There is no single source of truth. You will likely forget to update an entity and run into a stale state eventually. Therefore, the best practice is to store your state normalized so that each entity is a single source of truth. There will be no duplication of entities and thus no stale state when updating the one single source of truth, because each todo will reference to the updated `assignedTo` entity:

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
        name: "call it taming the state in react"
      }
    }
  },
  result: ["0", "1"]
}
~~~~~~~~

In conclusion, normalizing your state has two benefits. It keeps your state flat and thus easier manageable with immutable data structures. In addition, it groups entities to single sources of truth without any duplications. When you normalize your state, you will automatically get groupings of entities that could lead to their own reducers managing them. However, you don't want to start out with normalizing your state from the beginning. First you should try to introduce Redux itself to your application. Once you experience difficulities when altering your state, because it is deeply nested, or you experience issues updating single entities, because there is no single source of truth, you would want to introduce state normalization in your Redux application.

Before you will apply the normalization in your own Todo application as exercise, there exists yet another benefit when normalizing your state with a library such as normalizr. It is about denormalization: how do components retrieve the normalized state? You will learn it in the next chapter.

## Selectors

In Redux, there is the concept of selectors to retrieve derived properties from your state. A selector is a function that takes the state as argument and returns a substate or derived properties of it. It can be that they only return a substate of your global state or that they already preprocess your state to return derived properties. The function can take optional arguments to support the selection process.

### Plain Selectors

Selectors usually follow one syntax. The mandatory argument of a selector is the state from where it has to select from. There can be optional arguments that are in a supportive role to select the substate or the derived properties.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// selector
(state) => derived properties
// selector with optional arguments
(state, args) => derived properties
~~~~~~~~

Selectors are not mandatory. When thinking about all the parts in Redux, only the action(s), the reducer(s) and the Redux store are a binding requirement. Similar to action creators, selectors can be used to achieve an improved developer experience in a Redux architecture, but you don't need to use them. What does a selector look like? It is a plain function, as mentioned, that could live anywhere in your application. However, you would use it, maybe import it, when using Redux in React, in your `mapStateToProps()` function. So instead of retrieving the state explicitly:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function mapStateToProps(state) {
  return {
    todos: state.todoState,
  };
}
~~~~~~~~

You would retrieve it implicitly via a selector:

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function getTodos(state) {
  return state.todoState;
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

It is similar to the action and reducer concept. Instead of manipulating the state directly in the Redux store, you will use action(s) and reducer(s) to alter it indirectly. The same applies for selectors that don't retrieve the derived properties directly but indirectly from the global state.

You may wonder: Why are selectors an advantage? There are several benefits. A selector can be reused. You will run into cases where you select the derived properties or substate more often from the global state. For instance, you may have multiple places in your application where you want to show only completed todo items. And that's always a good sign to use a function in the first place. In addition, selectors can be tested separately. They are pure functions and thus an easily testable part in your application and the overall Redux architecture. Last but not least, deriving properties from state can become a complex undertaking in a scaling application. As mentioned, a selector could get optional arguments to derive more sophisticated properties from the state. The selector function itself would become more complex, but it would be encapsulated in one function rather than, for instance in React and Redux, in multiple `mapStateToProps()` functions.

### Denormalize State in Selectors

In the previous chapter about normalization, there was one benefit left unexplained. It is about selecting normalized state from your state layer to pass it to your view layer. Personally, I would argue a normalized state structure makes it much more convenient to select a substate from it. When you recall the normalized state structure, it looked something like the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  entities: ...
  ids: ...
}
~~~~~~~~

For instance, in a real work application it would look like the following:

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

If you recall the Redux in React chapter, there you passed the list of todos from the `TodoList` component, because it is a connected component, down to the whole component tree. How would you solve this with the normalized state from above?

Assuming that the reducer stored the state in a normalized immutable data structure, you would only pass the list of todo `ids` to your `TodoList` component. Because this component manages the list and not the entities themselves, it makes perfect sense that it only gets the list with references to the entities.

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
  return state.todoState.ids;
}

function mapStateToProps(state) {
  return {
    todosAsIds: getTodosAsIds(state),
  };
}

const ConnectedTodoList = connect(mapStateToProps)(TodoList);
~~~~~~~~

Now the `ConnectedTodoItem` component, that already passes the `onToggleTodo()` handler via the `mapDispatchToProps()` function to its plain `TodoItem` component, would select the todo entity matching to the incoming `todoId` property.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function getTodoAsEntity(state, id) {
  return state.todoState.entities[id];
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

const ConnectedTodoItem = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoItem);
~~~~~~~~

The `TodoItem` component itself would stay the same. It still gets the `todo` item and the `onToggleTodo()` handler as arguments. In addition, you can see two more concepts that were explained earlier. First, the selector grows in complexity because it gets optional arguments to select derived properties from the state. Second, the `mapStateToProps()` function makes use of the incoming props from the `TodoList` component that uses the `ConnectedTodoItem` component.

As you can see, the normalized state requires to use more connected components. More components are responsible to select their needed derived properties. But in a growing application, following this pattern can make it easier to reason about it. You only pass properties that are really necessary to the component. In the last case, the `TodoList` component only cares about a list of references yet the `TodoItem` component itself cares about the entity that is selected by using the reference passed down by the `TodoList` component.

There exists another way to denormalize your normalized state when using a library such as normalizr. The previous scenario allowed you to only pass the minimum of properties to the components. Each component was responsible to select its state. In the nexy scenario, you will denormalize your state in one component while the other components don't need to care about it. You will use the defined schema, which you have used for the initial normalization, to reverse the normalization.

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
  const entities = state.todoState.entities;
  const ids = state.todoState.ids;
  return denormalize(ids, [ todoSchema ], entities);
}

function mapStateToProps(state) {
  return {
    todos: getTodos(state),
  };
}

const ConnectedTodoList = connect(mapStateToProps)(TodoList);
~~~~~~~~

In this scenario, the whole normalized data structure gets denormalized in the selector. You will have the whole list of todos in your `TodoList` component. The `TodoItem` component wouldn't need to take care about the denormalization.

As you can see, there are two essential ways on how to deal with normalized state in your selectors or in general in the `mapStateToProps()` functions. It is up to you to find about the best suited implementation for your own use case. Perhaps you don't even need to normalize your state in the first place, because it is already flat or not very deeply nested.

### Reselect

When using selectors in a scaling application, you should consider a library called [reselect](https://github.com/reactjs/reselect) that provides you with advanced selectors. Basically, it uses the same concept of plain selectors as you have learned before, but comes with two improvements.

A plain selector has one constraint:

* *"Selectors can compute derived data, allowing Redux to store the minimal possible state."*

There are two more constraints when using selectors from the reselect library:

* *"Selectors are efficient. A selector is not recomputed unless one of its arguments change."*
* *"Selectors are composable. They can be used as input to other selectors."*

Selectors are pure functions without any side-effects. The output doesn't change when the input stays the same. Therefore, when a function is called twice and its arguments didn't change, it returns the same output. This proposition is used in reselect's selectors. It is called memoization in programming. A selector doesn't need to compute everything again when its input didn't change. It will simply return the same output, because it is a pure function. With memoization it remembers the previous input and if the input didn't change it returns the previous output. In a scaling application this can have a performance impacts.

Another benefit, when using reselect, is the ability to compose selectors. It supports the case of implementing reusable selectors that only solve one problem. Afterward they can be composed in a functional programming style.

The book will not dive deeper into the reselect library. When learning Redux it is good to know about these advanced selectors, but you are fine by using plain selectors in the beginning. If you cannot stay put, you can read up the example usages in the [official GitHub repository](https://github.com/reactjs/reselect) and apply in your projects while reading the book.

## Hands On: Todo with Advanced Redux

Now in the Todo application, you can refactor everything to use the advanced techniques you have learned in the previous chapters: a middleware, an immutable data structure using JavaScript spread operators, a normalized data structure and selectors. Let's continue with the Todo application that you have build when you connected React and Redux. The last version can be found in this [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/3.0.0).

In the first part, let's use the [redux-logger](https://github.com/evgenyrodionov/redux-logger) middleware. Therefore, you have to install it on the command line:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save redux-logger
~~~~~~~~

Next you can use it when you create your store:

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import { applyMiddleware, combineReducers, createStore } from 'redux';
# leanpub-end-insert
import { Provider, connect } from 'react-redux';
# leanpub-start-insert
import { createLogger } from 'redux-logger';
# leanpub-end-insert
import './index.css';

...

// store

const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
});

# leanpub-start-insert
const logger = createLogger();

const store = createStore(
  rootReducer,
  undefined,
  applyMiddleware(logger)
);
# leanpub-end-insert
~~~~~~~~

When you start your Todo application now, you should see the output of the `logger` in the developer console of your browser when dispatching actions. The Todo application with the middleware using redux-logger can be found in this [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/4.0.0).

The second part of this chapter is using the JavaScript spread operators instead of the `Object.assign()` function to keep an immutable data structure. You can apply it in your reducer functions:

{title="src/index.js",lang="javascript"}
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
# leanpub-end-insert
      : todo
  );
}
~~~~~~~~

The application should work the same as before, but this time with the spread operator for keeping an immutable data structure and thus an immutable state object. The source code can be found again in this [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/5.0.0).

In the third part of applying the advanced techniques from the previous chapters, you will use a normalized state structure. Therefore, you can install the neat library [normalizr](https://github.com/paularmstrong/normalizr) on the command line:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save normalizr
~~~~~~~~

Let's have a look at the initial state for the `todoReducer`. You can make up an initial state for them. For instance, what about completing all coding examples in this book by having todo items for them?

{title="src/index.js",lang="javascript"}
~~~~~~~~
const todos = [
  { id: '1', name: 'Redux Standalone with advanced Actions' },
  { id: '2', name: 'Redux Standalone with advanced Reducers' },
  { id: '3', name: 'Bootstrap App with Redux' },
  { id: '4', name: 'Naive Todo with React and Redux' },
  { id: '5', name: 'Sophisticated Todo with React and Redux' },
  { id: '6', name: 'Connecting State Everywhere' },
  { id: '7', name: 'Todo with advanced Redux' },
  { id: '8', name: 'Todo but more Features' },
  { id: '9', name: 'Todo with Notifications' },
  { id: '10', name: 'Hacker News with Redux' },
];

function todoReducer(state = todos, action) {
  ...
}
~~~~~~~~

You can use normalizr to normalize this data structure. First, you have to define a schema:

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { applyMiddleware, combineReducers, createStore } from 'redux';
import { Provider, connect } from 'react-redux';
import { createLogger } from 'redux-logger';
# leanpub-start-insert
import { schema, normalize } from 'normalizr';
# leanpub-end-insert
import './index.css';

# leanpub-start-insert
// schemas

const todoSchema = new schema.Entity('todo');
# leanpub-end-insert
~~~~~~~~

Second, you can use the schema to normalize your initial todos and use them as default parameter in your `todoReducer`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
// reducers

const todos = [
  ...
];

# leanpub-start-insert
const normalizedTodos = normalize(todos, [todoSchema]);
const initialTodoState = {
  entities: normalizedTodos.entities.todo,
  ids: normalizedTodos.result,
};

function todoReducer(state = initialTodoState, action) {
# leanpub-end-insert
  ...
}
~~~~~~~~

Third, your `todoReducer` needs to handle the normalized state structure. It has to deal with entities and a result (list of ids). You can output the normalized todos even though the Todo application crashes when you attempt to start it.

{title="src/index.js",lang="javascript"}
~~~~~~~~
const normalizedTodos = normalize(todos, [todoSchema]);
console.log(normalizedTodos);
~~~~~~~~

The adjusted reducer would have the following internal functions:

{title="src/index.js",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
  const todo = { ...action.todo, completed: false };
  const entities = { ...state.entities, [todo.id]: todo };
  const ids = [ ...state.ids, action.todo.id ];
  return { ...state, entities, ids };
}

function applyToggleTodo(state, action) {
  const id = action.todo.id;
  const todo = state.entities[id];
  const toggledTodo = { ...todo, completed: !todo.completed };
  const entities = { ...state.entities, [id]: toggledTodo };
  return { ...state, entities };
}
~~~~~~~~

It operates on `entities` and `ids`, because these are the output from the normalization. Last but not least, when connecting Redux with React, the components need to be aware of the normalized data structure. First, the connection between store and components:

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function mapStateToPropsList(state) {
  return {
    todosAsIds: state.todoState.ids,
  };
}

function mapStateToPropsItem(state, props) {
  return {
     todo: state.todoState.entities[props.todoId],
  };
}
# leanpub-end-insert

# leanpub-start-insert
function mapDispatchToPropsItem(dispatch) {
# leanpub-end-insert
  return {
     onToggleTodo: id => dispatch(doToggleTodo(id)),
  };
}

# leanpub-start-insert
const ConnectedTodoList = connect(
  mapStateToPropsList
)(TodoList);
const ConnectedTodoItem = connect(
  mapStateToPropsItem,
  mapDispatchToPropsItem
)(TodoItem);
# leanpub-end-insert
~~~~~~~~

Second, the `TodoList` component receives only the `todosAsIds` and the `TodoItem` receives the `todo` entity.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoApp() {
  return <ConnectedTodoList />;
}

# leanpub-start-insert
function TodoList({ todosAsIds }) {
# leanpub-end-insert
  return (
    <div>
# leanpub-start-insert
      {todosAsIds.map(todoId => <ConnectedTodoItem
        key={todoId}
        todoId={todoId}
      />)}
# leanpub-end-insert
    </div>
  );
}

function TodoItem({ todo, onToggleTodo }) {
  ...
}
~~~~~~~~

The application should work again. Start it and play around with it. You can find the source code in the [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/6.0.0). You have normalized your initial state structure and adjusted your reducer to deal with the new data structure.

In the fourth and last part, you are going to use selectors for your Redux architecture. This refactoring is fairly straight forward. You have to extract the parts that operate on the state in your `mapStateToProps()` functions to selector functions. First, define the selector functions:

{title="src/index.js",lang="javascript"}
~~~~~~~~
// selectors

function getTodosAsIds(state) {
  return state.todoState.ids;
}

function getTodo(state, todoId) {
  return state.todoState.entities[todoId];
}
~~~~~~~~

Second, you can use these functions instead of operating on the state directly in your `mapStateToProps()` functions:

{title="src/index.js",lang="javascript"}
~~~~~~~~
// Connecting React and Redux

function mapStateToPropsList(state) {
  return {
# leanpub-start-insert
    todosAsIds: getTodosAsIds(state),
# leanpub-end-insert
  };
}

function mapStateToPropsItem(state, props) {
  return {
# leanpub-start-insert
     todo: getTodo(state, props.todoId),
# leanpub-end-insert
  };
}
~~~~~~~~

The Todo application should work with selectors now. You can find it in the [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/7.0.0) again.

## Hands On: Todo but more Features

In the Todo application, there are two pieces missing feature-wise: the ability to add a todo and to filter todos by their complete state. Let's begin with the creation of a todo item. First, there needs to be a component where you can type in a todo name and execute its creation.

{title="src/index.js",lang="javascript"}
~~~~~~~~
class TodoCreate extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      value: '',
    };

    this.onCreateTodo = this.onCreateTodo.bind(this);
    this.onChangeName = this.onChangeName.bind(this);
  }

  onChangeName(event) {
    this.setState({ value: event.target.value });
  }

  onCreateTodo(event) {
    this.props.onAddTodo(this.state.value);
    this.setState({ value: '' });
    event.preventDefault();
  }

  render() {
    return (
      <div>
        <form onSubmit={this.onCreateTodo}>
          <input
            type="text"
            placeholder="Add Todo..."
            value={this.state.value}
            onChange={this.onChangeName}
          />
          <button type="submit">Add</button>
        </form>
      </div>
    );
  }
}
~~~~~~~~

Notice again that the component is completely unaware of Redux. It only updates its local `value` state and once the form gets submitted, it uses the local `value` state for the `onAddTodo()` callback function that's accessible in the `props` object. The component doesn't know whether the callback function updates the local state of a parent component or the Redux store. Next, you can use the connected version of this component in the `TodoApp` component.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoApp() {
  return (
# leanpub-start-insert
    <div>
      <ConnectedTodoCreate />
# leanpub-end-insert
      <ConnectedTodoList />
# leanpub-start-insert
    </div>
# leanpub-end-insert
  );
}
~~~~~~~~

The last step is to wire the React component to the Redux store by making it a connected component in the first place.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function mapDispatchToPropsCreate(dispatch) {
  return {
    onAddTodo: name => dispatch(doAddTodo(uuid(), name)),
  };
}

const ConnectedTodoCreate = connect(null, mapDispatchToPropsCreate)(TodoCreate);
~~~~~~~~

It uses the `mapDispatchToPropsCreate()` function to get access to the dispatch method of the Redux store. The `doAddTodo()` action creator takes the name of the todo item, coming from the `TodoCreate` component, and generates a unique identifier with the `uuid()` function. The `uuid()` function is a neat little helper function that comes from the [uuid](https://github.com/kelektiv/node-uuid) library. First, you have to install it:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save uuid
~~~~~~~~

And second, you can import it to generate unique identifiers for you:

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { applyMiddleware, combineReducers, createStore } from 'redux';
import { Provider, connect } from 'react-redux';
import { createLogger } from 'redux-logger';
import { schema, normalize } from 'normalizr';
# leanpub-start-insert
import uuid from 'uuid/v4';
# leanpub-end-insert
import './index.css';
~~~~~~~~

You can try to create a todo item in your Todo application now. It should work. Next you want to make use of your filter functionality to filter the list of todo items by their `completed` property. First, you have to add a `Filter` component.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function Filter({ onSetFilter }) {
  return (
    <div>
      Show
      <button
        type="button"
        onClick={() => onSetFilter('SHOW_ALL')}>
        All</button>
      <button
        type="button"
        onClick={() => onSetFilter('SHOW_COMPLETED')}>
        Completed</button>
      <button
        type="button"
        onClick={() => onSetFilter('SHOW_INCOMPLETED')}>
        Incompleted</button>
    </div>
  );
}
~~~~~~~~

The `Filter` component only receives a callback function. Again it doesn't know anything about the state management that is happening above in Redux or somewhere else. The callback function is only used in different buttons to set specific filter types. You can use the connected component in the `TodoApp` component again.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoApp() {
  return (
    <div>
# leanpub-start-insert
      <ConnectedFilter />
# leanpub-end-insert
      <ConnectedTodoCreate />
      <ConnectedTodoList />
    </div>
  );
}
~~~~~~~~

Last but not least, you have to connect the `Filter` component to actually use it in the `TodoApp` component. It dispatched the `doSetFilter` action creator by passing the filter type from the underlying buttons in the `Filter` component.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function mapDispatchToPropsFilter(dispatch) {
  return {
    onSetFilter: filterType => dispatch(doSetFilter(filterType)),
  };
}

const ConnectedFilter = connect(null, mapDispatchToPropsFilter)(Filter);
~~~~~~~~

When you start your Todo application now, you will see that the `filterState` will change once you click on one of your filter buttons. But nothing happens to your displayed todos. They are not filtered and that's because in your selector you select the whole list of todos. The next step would be to adjust the selector to only select the todos in the list that are matching the filter. First, you can define filter functions that match todos according to their `completed` state.

{title="src/index.js",lang="javascript"}
~~~~~~~~
// filters

const VISIBILITY_FILTERS = {
  SHOW_COMPLETED: item => item.completed,
  SHOW_INCOMPLETED: item => !item.completed,
  SHOW_ALL: item => true,
};
~~~~~~~~

Second, you can use your selector to only select the todos matching a filter. You already have all selectors in place. But you need to adjust one of them to filter the todos according to the `filterState` from the Redux store.

{title="src/index.js",lang="javascript"}
~~~~~~~~
// selectors

function getTodosAsIds(state) {
# leanpub-start-insert
  return state.todoState.ids
    .map(id => state.todoState.entities[id])
    .filter(VISIBILITY_FILTERS[state.filterState])
    .map(todo => todo.id);
# leanpub-end-insert
}

function getTodo(state, todoId) {
  return state.todoState.entities[todoId];
}
~~~~~~~~

Since your state is normalized, you have to map through all your `ids` to get a list of `todos`, filter them by `filterState`, and map them back to 'ids'. That's a tradeoff you are going when normalizing your data structure, because you always have to denormalize it at some point. Your filter functionality should work once you start your application again.

You can find the final application in this [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/8.0.0). It applies all the learnings about the Redux middleware, immutable and normalized data structures and selectors.
