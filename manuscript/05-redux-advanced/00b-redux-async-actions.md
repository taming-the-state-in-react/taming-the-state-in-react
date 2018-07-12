# Asynchronous Redux

In the book, you have only used synchronous actions so far. There is no delay of the action dispatching involved. Yet, sometimes you want to delay an action. The example can be a simple one: Imagine you want to have a notification for your application user when a todo item was created. The notification needs to show up, but also should hide after one second. The first action would show the notification, because it sets a `isShowingNotification` property to true in the Redux store. Afterward, you would want to have a delayed second action to hide the notification again. In the simplest case, it would look like the following:

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch({ type: 'NOTIFICATION_SHOW', text: 'Todo created.' });
setTimeout(() => {
  store.dispatch({ type: 'NOTIFICATION_HIDE' })
}, 1000);
~~~~~~~~

There is nothing against a simple `setTimeout()` function in your application. Sometimes it is easier to remember the basics in JavaScript than trying to apply yet another library to fix a problem. Since you know about actions creators, the next step could be to extract these actions into according action creators and action types.

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

There are two problems in a growing application now. First, you would have to duplicate the logic of the delayed action with the `setTimeout()` every time you want to show a notification and hide it again. What could be helpful in this case? Right, extracting the functionality as a function. Second, once your application dispatches multiple notifications at various places at the same time, the first running action that hides a notification would hide all of them. What could be helpful in this case? Correct, you need to give each notification an identifier.

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

Now each notification can be identified in the reducer and thus be either shown or hidden. The extracted function gets control over the `dispatch()` method from the Redux store, because it needs to dispatch a delayed action after all.

**Why not passing the Redux store instead?** Usually, you want to avoid to directly pass the store around. You have encountered the same reasoning in the book when weaving the Redux store for the first time into your React application. You want to make the functionalities of the store available, but not the entire store itself. That's why you only have the `dispatch()` method and not the entire store in your `mapDispatchToProps()` function when using react-redux. A connected component does never have access to the store itself and thus no other functionalities should have direct access to it to keep up with those constraints.

The pattern from above suffices for simple Redux applications that need a delayed (asynchronous) action. However, in scaling applications it has a drawback. The approach creates a second type of action. While there are synchronous actions that can be dispatched directly, as you have used them before, there are pseudo asynchronous actions, too. These pseudo asynchronous actions cannot be dispatched directly, but are used within a function that accepts the dispatch method as argument to dispatch actions eventually. That's what you have implemented with the `showNotificationWithDelay()` function now. Wouldn't it be great to use both types of actions the same way without worrying to pass around the dispatch method and without worrying about asynchronous or synchronous actions? You will find out about it in the next chapter.

## Redux Thunk

The previous question led Dan Abramov, the creator of Redux, thinking about a general pattern to the problem of asynchronous actions. He came up with the library called [redux-thunk](https://github.com/gaearon/redux-thunk) to legitimize the concept: Synchronous and asynchronous actions should be dispatched in a similar way from a Redux store. The redux-thunk library is used as middleware in your Redux store.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { createStore, applyMiddleware } from 'redux';
# leanpub-start-insert
import thunk from 'redux-thunk';
# leanpub-end-insert

...

const store = createStore(
  rootReducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(thunk)
# leanpub-end-insert
);
~~~~~~~~

Basically, it creates a middleware for your actions creators. In this middleware, you are enabled to dispatch asynchronous actions. Apart from dispatching objects, you can dispatch functions with Redux Thunk too. Before you have always dispatched objects as actions in this book. An action itself is an object and an action creator returns an action object.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// with plain action
store.dispatch({ type: 'NOTIFICATION_SHOW', text: 'Todo created.' });

// with action creator
function doShowNotification(text) {
  return {
    type: NOTIFICATION_SHOW,
    text,
  };
}

store.dispatch(doShowNotification('Todo created.'));
~~~~~~~~

However, now you can pass the dispatch method a function, too. The function will always give you access to the dispatch method again.

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

The dispatch method of the Redux store when using Redux Thunk will differentiate between passed objects and functions. The passed function is called a **thunk**. You can dispatch as many actions synchronously and asynchronously as you want in a thunk. When a thunk is growing and handles complex business logic at some point in your application, it is called a **fat thunk**. You can extract a thunk function as an asynchronous action creator, that is a higher-order function and returns the thunk function, and can be dispatched the same way as a synchronous action creator.

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

It is similar to the solution without Redux Thunk. However, this time you don't have to pass around the dispatch method and instead have access to it in the returned thunk function. Now, when using it in a React component, the component still only executes a callback function that it receives via its props. The connected component then dispatches an action, regardless of the action being synchronously or asynchronously, in the `mapDispatchToProps()` function.

That are the basics of Redux Thunk. There are a few more things that are good to know about:

* **getState():** A thunk function gives you the `getState()` method of the Redux store as second argument: `function (dispatch, getState)`. However, you should generally avoid it. It's best practice to pass all necessary state to the action creator instead of retrieving it in the thunk.
* **Promises:** Thunks work great in combination with promises. You can return a promise from your thunk and use it, for instance, to wait for its completion: `store.dispatch(showNotificationWithDelay('Todo created.')).then(...)`.
* **Recursive Thunks:** The dispatch method in a thunk can again be used to dispatch an asynchronous action. Thus, you can apply the thunk pattern recursively.

### Hands On: Todo with Notifications

After learning about asynchronous actions, the Todo application could make use of notifications, couldn't it? You can continue with the Todo application that you have built in the last chapters. As an alternative, you can clone it from this [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/8.0.0).

The first part of this hands on chapter is a great repetition on using everything you have learned before asynchronous actions. First, you have to implement a notification reducer which evaluates all actions that should generate a notification. In this case, the action that creates a todo item should be reused for the notification. That's the great thing about Redux: All reducers who care about an action can make use of it. Also the `id` of the todo item can be used to store the notification with its own identifier to hide it later again.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function notificationReducer(state = {}, action) {
  switch(action.type) {
    case TODO_ADD : {
      return applySetNotifyAboutAddTodo(state, action);
    }
    default : return state;
  }
}

function applySetNotifyAboutAddTodo(state, action) {
  const { name, id } = action.todo;
  return { ...state, [id]: 'Todo Created: ' + name  };
}
~~~~~~~~

Second, you have to include the reducer in your combined reducer to make it accessible to the Redux store.

{title="src/index.js",lang="javascript"}
~~~~~~~~
const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
# leanpub-start-insert
  notificationState: notificationReducer,
# leanpub-end-insert
});
~~~~~~~~

The Redux part is done. It is only a reducer that you need to include in the Redux store. The action gets reused. Third, you have to implement a React component that displays all of your notifications.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function Notifications({ notifications }) {
  return (
    <div>
      {notifications.map(note => <div key={note}>{note}</div>)}
    </div>
  );
}
~~~~~~~~

Fourth, you can include the connected version of the `Notifications` component in your `TodoApp` component.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoApp() {
  return (
    <div>
      <ConnectedFilter />
      <ConnectedTodoCreate />
      <ConnectedTodoList />
# leanpub-start-insert
      <ConnectedNotifications />
# leanpub-end-insert
    </div>
  );
}
~~~~~~~~

Last but not least, you have to wire up React and Redux in the connected `ConnectedNotifications` component.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function mapStateToPropsNotifications(state, props) {
  return {
     notifications: getNotifications(state),
  };
}

const ConnectedNotifications = connect(mapStateToPropsNotifications)(Notifications);
~~~~~~~~

The only thing left is to implement the missing selector `getNotifications()`. Since the notifications in the Redux store are saved as an object, because they are a map of identifier and notification pairs, you have to use a helper function to convert it into an array. You can extract the helper function, because you might need such functionalities more often and shouldn't couple it to the domain of notifications.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function getNotifications(state) {
  return getArrayOfObject(state.notificationState);
}

function getArrayOfObject(object) {
  return Object.keys(object).map(key => object[key]);
}
~~~~~~~~

The first part of this chapter is done. You should see a notification in your Todo application once you create a todo item. The second part will implement a `NOTIFICATION_HIDE` action and use it in the `notificationReducer` to remove the notification from the state again. First, you have to introduce the action type:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const TODO_ADD = 'TODO_ADD';
const TODO_TOGGLE = 'TODO_TOGGLE';
const FILTER_SET = 'FILTER_SET';
# leanpub-start-insert
const NOTIFICATION_HIDE = 'NOTIFICATION_HIDE';
# leanpub-end-insert
~~~~~~~~

Second, you can implement an action creator that uses the action type. It will hide (remove) the notification by id, because they are stored by id in the Redux store:

{title="src/index.js",lang="javascript"}
~~~~~~~~
function doHideNotification(id) {
  return {
    type: NOTIFICATION_HIDE,
    id
  };
}
~~~~~~~~

Third, you can capture it in the new `notificationReducer`. The JavaScript destructuring functionality can be used to omit a property from an object. You can simply omit the notification and return the remaining object. It is a neat trick if you want to get rid of a property in an object when knowing its key.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function notificationReducer(state = {}, action) {
  switch(action.type) {
    case TODO_ADD : {
      return applySetNotifyAboutAddTodo(state, action);
    }
# leanpub-start-insert
    case NOTIFICATION_HIDE : {
      return applyRemoveNotification(state, action);
    }
# leanpub-end-insert
    default : return state;
  }
}

# leanpub-start-insert
function applyRemoveNotification(state, action) {
  const {
    [action.id]: notificationToRemove,
    ...restNotifications,
  } = state;
  return restNotifications;
}
# leanpub-end-insert
~~~~~~~~

That was the second part of this chapter that introduced the hiding notification functionality. But you don't make any use of the functionality yet. The third and last part of this chapter will introduce asynchronous actions to hide a notification after a couple of seconds. As mentioned earlier, you wouldn't need a library to solve this problem. You could simply build up on the JavaScript `setTimeout()` functionality. But for the sake of learning about asynchronous actions in Redux, you will use Redux Thunk. It's up to you to exchange it with another asynchronous actions library afterward for the sake of learning about the alternatives. You will hear about these alternative libraries later.

First, you have to install the [redux-thunk](https://github.com/gaearon/redux-thunk) library on the command line:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save redux-thunk
~~~~~~~~

Second, you can import it in your code:

{title="src/index.js",lang="javascript"}
~~~~~~~~
...
# leanpub-start-insert
import thunk from 'redux-thunk';
# leanpub-end-insert
...
~~~~~~~~

And third, use it in your Redux store middleware:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const store = createStore(
  rootReducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(thunk, logger)
# leanpub-end-insert
);
~~~~~~~~

The application should still work. When using Redux Thunk, you can dispatch action objects as you did before. However, now you can dispatch thunks (functions), too. Rather than dispatching an action object that only creates a todo item, you can dispatch a thunk function that creates a todo item and hides the notification about the creation after a couple of seconds. You have two plain actions creators, `doAddTodo()` and `doHideNotification()`, already in place. You only have to use them in your thunk function.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function doAddTodoWithNotification(id, name) {
  return function (dispatch) {
    dispatch(doAddTodo(id, name));

    setTimeout(function () {
      dispatch(doHideNotification(id));
    }, 5000);
  }
}
~~~~~~~~

In the last step, you have to use the `doAddTodoWithNotification()` rather than the `doAddTodo()` action creator when connecting Redux and React in your `TodoCreate` component.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function mapDispatchToPropsCreate(dispatch) {
  return {
# leanpub-start-insert
    onAddTodo: name => dispatch(
      doAddTodoWithNotification(uuid(), name)
    ),
# leanpub-end-insert
  };
}
~~~~~~~~

That's it. Your notifications should work and hide after five seconds now. Basically, you have built the foundation for a notification system in your Todo application. You can use it for other actions, too. For instance, when completing a todo item you could trigger a notification too. The project can be found again in this [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/9.0.0).

## Asynchronous Actions Alternatives

The whole concept around asynchronous actions led to a handful of libraries which solve this particular issue. Redux Thunk was only the first one introduced by Dan Abramov. However, he agrees that there are use cases where Redux Thunk doesn't solve the problem in an efficient way. Redux Thunk can be used when you encounter a use case for asynchronous actions for the first time. But when there are more complex scenarios involved, you can use advanced solutions beyond Redux Thunk.

All of these solutions address the problem as side-effects in Redux. Asynchronous actions are used to deal with those side-effects. They are most often used when performing impure operations which happen to be often asynchronous too: fetching data from an API, delaying an execution (e.g. hiding a notification), or accessing the browser cache. All these operations are asynchronous and impure, hence are solved with asynchronous actions. In an application, that uses the functional programming paradigm, you want to have all these impure operations on the edge of your application. You don't want to have these close to your core application.

This chapter briefly shows the alternatives that you can use instead of Redux Thunk. Among all the different alternatives, I only want to introduce you to the most popular and innovative ones.

### Redux Saga

[Redux Saga](https://github.com/redux-saga/redux-saga) is the most popular asynchronous actions library for Redux. *"The mental model is that a saga is like a separate thread in your application that's solely responsible for side effects."* Basically, it outsources the impure operations into separate threads. These threads can be started, paused or cancelled with plain Redux actions from your core application. Thereby, threads in Redux Saga make it simple to keep your side-effects away from your core application. However, threads can dispatch actions and have access to the state though.

Redux Saga uses [JavaScript ES6 Generators](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Generator) as underlying technology. That's why the code reads like synchronous code. You avoid it having callback functions. The advantage over Redux Thunk is that your actions stay pure and thus they can be tested well.

Apart from Redux Thunk and Redux Saga, there are other side-effect libraries for Redux. If you want to try out observables in JavaScript, you could give [Redux Observable](https://github.com/redux-observable/redux-observable) a shot. It builds up on RxJS, a generic library for reactive programming. If you are interested in another library that uses reactive programming principles, too, you can try [Redux Cycles](https://github.com/cyclejs-community/redux-cycles).

In conclusion, as you can see, all these libraries, Redux Saga, Redux Observable and Redux Cycles, make use of different techniques in JavaScript. You can give them a try to experiment with recent JavaScript technologies: generators or observables. The whole ecosystem around asynchronous actions is a great playground to try new things in JavaScript in the end.

### Hands On: Todo with Redux Saga

In a previous chapter, you used Redux Thunk to dispatch asynchronous actions. These were used to add a todo item with a notification whereas the notification vanishes after a couple of seconds again. In this chapter you will use Redux Saga instead of Redux Thunk. Therefore, you can install the former library and uninstall the latter one. You can continue with your previous Todo application.

{title="Command Line: /",lang="text"}
~~~~~~~~
npm uninstall --save redux-thunk
npm install --save redux-saga
~~~~~~~~

The action you have used before with a thunk becomes a pure action creator now. It will be used to start the saga thread.

{title="src/index.js",lang="javascript"}
~~~~~~~~
const TODO_ADD_WITH_NOTIFICATION = 'TODO_ADD_WITH_NOTIFICATION';

...

function doAddTodoWithNotification(id, name) {
  return {
    type: TODO_ADD_WITH_NOTIFICATION,
    todo: { id, name },
  };
}
~~~~~~~~

Now you can introduce your first saga that listens on this particular action, because the action is solely used to start the saga thread.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import { takeEvery } from 'redux-saga/effects';

...

// sagas

function* watchAddTodoWithNotification() {
  yield takeEvery(TODO_ADD_WITH_NOTIFICATION, ...);
}
~~~~~~~~

Most often you will find one part of the saga watching incoming actions and evaluating them. If an evaluation applies truthfully, it will often call another generator function, identified with the asterisk, that handles the side-effect. That way, you can keep your side-effect watcher maintainable and don't clutter them with business logic. In the previous example, a `takeEvery()` effect of Redux Saga is used to handle every action with the specified action type. Yet there are other effects in Redux Saga such as `takeLatest()` which only takes the last of the incoming actions by action type.

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { delay } from 'redux-saga';
import { put, takeEvery } from 'redux-saga/effects';
# leanpub-end-insert

function* watchAddTodoWithNotification() {
# leanpub-start-insert
  yield takeEvery(TODO_ADD_WITH_NOTIFICATION, handleAddTodoWithNotification);
# leanpub-end-insert
}

# leanpub-start-insert
function* handleAddTodoWithNotification(action) {
  const { todo } = action;
  const { id, name } = todo;
  yield put(doAddTodo(id, name));
  yield delay(5000);
  yield put(doHideNotification(id));
}
# leanpub-end-insert
~~~~~~~~

As you can see, in JavaScript generators you use the `yield` statement to execute asynchronous code synchronously. Only when the function after the `yield` resolves, the code will execute the next line of code. Redux Saga comes with helper functions such as `delay()` that can be used to delay the execution by an amount of time. It would be the same as using `setTimeout()` in JavaScript, but the `delay()` helper makes it more concise when using JavaScript generators and can be used in a synchronous way when using the yield statement.

In the end, you only have to exchange your middleware in your Redux store from using Redux Thunk to Redux Saga.

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import createSagaMiddleware, { delay } from 'redux-saga';
# leanpub-end-insert
import { put, takeEvery } from 'redux-saga/effects';

...

const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
  notificationState: notificationReducer,
});

const logger = createLogger();
# leanpub-start-insert
const saga = createSagaMiddleware();
# leanpub-end-insert

const store = createStore(
  rootReducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(saga, logger)
# leanpub-end-insert
);

# leanpub-start-insert
saga.run(watchAddTodoWithNotification);
# leanpub-end-insert
~~~~~~~~

That's it. You Todo application should run with Redux Saga instead of Redux Thunk now. The final application can be found in the [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/10.0.0) again. In the future, it is up to you to decide on an asynchronous actions library when using Redux. Is it Redux Thunk or Redux Saga? Or do you decide to try something new with Redux Observable or Redux Cycles? The Redux ecosystem is full of amenities to try cutting edge JavaScript features such as generators or observables. However, you need to decide yourself if you need a asynchronous actions library in the first place.
