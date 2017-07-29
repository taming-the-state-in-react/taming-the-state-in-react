# Asynchronours Redux

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

**Why not passing the Redux store instead?** Usually you want to avoid to pass the store around directly. You have encountered the same reasoning in the book when weaving the Redux store for the first time into your React application. You want to make the functionalities of the store available, but not the entire store itself. That's why you only have the `dispatch()` method and not the entire store in your `mapDispatchToProps()` function when using react-redux. A connected component has never access to the store directly and thus no other functionalities should have direct access to it.

The pattern from above suffices for simple Redux applications that need a delayed action. However, in scaling applications it has a drawback. The approach creates two types of action creators. While there are synchronours action creators that can be dispacthed directly, there are pseudo asynchronours action creators too. These pseudo asynchronours action creators cannot be dispacthed directly but have to accept the dispatch method as argument. Wouldn't it be great to use both types of actions the same without worrying to pass around the dispatch method and any asynchronours or synchronours behavior?

## Redux Thunk

- TODO https://www.reddit.com/r/reactjs/comments/6e0vgt/is_my_understanding_of_reduxthunk_correct/?st=1Z141Z3&sh=255adc41
- TODO https://medium.com/@talkol/redux-thunks-dispatching-other-thunks-discussion-and-best-practices-dd6c2b695ecf
- TODO https://decembersoft.com/posts/what-is-the-right-way-to-do-asynchronous-operations-in-redux/

The previous question led Dan Abramov, the creator of Redux, thinking about a general pattern to the problem of asynchronours actions. He came up with the library called [redux-thunk](https://github.com/gaearon/redux-thunk) to legitimize the concept. Synchronours and asynchronours action creators should be dispatched in a similar way from a Redux store. It is used as middleware in your Redux store.

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

### Hands On: Todo with Notifications

After learning about asynchronous actions, the Todo application could make use of notifications, couldn't it? The first part of this hands on chapter is a great repition on using everything you have learned before asynchronrsous actions. First, you have to implement a notification reducer that evaluates actions that should generate a notification.

{title="Code Playground",lang="javascript"}
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

You don't need to create a new action type. Instead you can reuse the action you already have to add todos. When a todo gets created, the notification reducer will store a new notification about the created todo item. Second, you have to include the reducer in your combined reducer to make it accessible to the Redux store.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
# leanpub-start-insert
  notificationState: notificationReducer,
# leanpub-end-insert
});
~~~~~~~~

The Redux part is done. It is only a reducer and including it in the Redux store. The action gets reused. Third, you have to implement a React component that displays all of your notifications.

{title="Code Playground",lang="javascript"}
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

{title="Code Playground",lang="javascript"}
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

{title="Code Playground",lang="javascript"}
~~~~~~~~
function mapStateToPropsNotifications(state, props) {
  return {
     notifications: getNotifications(state),
  };
}

const ConnectedNotifications = connect(mapStateToPropsNotifications)(Notifications);
~~~~~~~~

The only thing left is to implement the missing selector `getNotifications()`. Since the notifications in the Redux store as saved as an object, you have to use a helper function to convert it into an array. It is good to extract the helper function earlier on, because you might need such functionalities more often and shouldn't couple it to the domain of notifications.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function getNotifications(state) {
  return getArrayOfObject(state.notificationState);
}

function getArrayOfObject(object) {
  return Object.keys(object).map(key => object[key]);
}
~~~~~~~~

The first part of the "Hands On" chapter is done. You should see a notification in your Todo application once you create a todo item. The second part will implement a `NOTIFICATION_HIDE` action and use it in the `notificationReducer` to remove the notification from the state. First, you have to introduce the action type:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const NOTIFICATION_HIDE = 'NOTIFICATION_HIDE';
~~~~~~~~

Second, you can implement an action creator that uses the action type. It will hide (remove) the notification by id, because they are stored by id:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doHideNotification(id) {
  return {
    type: NOTIFICATION_HIDE,
    id
  };
}
~~~~~~~~

Third, you can capture it in the `notificationReducer`. The JavaScript destructuring functionality can be used to omit a property from an object. You can simply omit the notification and return the remaining object.

{title="Code Playground",lang="javascript"}
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

That was the second part of the "Hands On" chapter that introduced the hiding notification functionality. The third and last part of the "Hands On" chapter will introduce asynchronous actions to hide a notification after a couple of seconds. As mentioned earlier, you wouldn't need a library to solve this problem. You could simply built on the JavaScript timeout functionality. But for the sake of learning about asynchronous actions, you will use Redux Thunk. It's up to you to exchange it with another asynchronours actions library afterward for the sake of learning about the alternatives.

First, you have to install the [redux-thunk](https://github.com/gaearon/redux-thunk) on the command line:

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux-thunk
~~~~~~~~

Second, you can import it in your code:

{title="Code Playground",lang="javascript"}
~~~~~~~~
...
# leanpub-start-insert
import thunk from 'redux-thunk';
# leanpub-end-insert
...
~~~~~~~~

And third, use it in your Redux store middleware:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(
  rootReducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(thunk, logger)
# leanpub-end-insert
);
~~~~~~~~

The application should still work. When using Redux Thunk, you can dispatch action objects as before. However, now you can dispatch thunks (functions) too. Rather than dispatching an action object that only creates a todo item, you can dispatch a thunk function that creates a todo item and hides the notification about the creation after a couple of seconds. You have two plain actions creators, `doAddTodo()` and `doHideNotification()`, already in place. You only have to reuse it in your thunk function.

{title="Code Playground",lang="javascript"}
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

{title="Code Playground",lang="javascript"}
~~~~~~~~
function mapDispatchToPropsCreate(dispatch) {
  return {
# leanpub-start-insert
    onAddTodo: name => dispatch(doAddTodoWithNotification(uuid(), name)),
# leanpub-end-insert
  };
}
~~~~~~~~

That's it. Your notifications should work and hide after five seconds. Basically you have built the foundation for a notifiaction system in your Todo application. You can use it for other actions too. The project can be found in the [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/9.0.0).

## Asynchornous Actions Alternatives

The whole concept around asynchronours actions led to a handful of libraries which solve this particular issue. Redux Thunk was only the first one introduced by Dan Abramov. However, he agrees that there are use cases where Redux Thunk doesn't solve the problem in an efficient way. Redux Thunk can be used when you encounter the first time an use case for asynchronours actions. But when there are more complex scenarios involved, you can use advanced solutions beyond Redux Thunk.

All of these solutions address the problem as side-effects in Redux. Asynchronours actions are used to deal with those side effects. They are most often used when performing impure operations: fetching data from an API, delaying an execution or accessing the browser cache. All these operations are asynchronous and impure, hence are solved with asynchronous actions. In an application, that uses the functional programming paradigm, you want to have all these impure operations on the edge of your application. You don't want to have these close to your core application.

This chapter briefly shows the alternatives that you could use instead of Redux Thunk. Among all the different alternatives, I only want to introduce you to the most popular and innovative ones.

### Redux Saga

[Redux Saga](https://github.com/redux-saga/redux-saga) is the most popular asynchronous actions library for Redux. *"The mental model is that a saga is like a separate thread in your application that's solely responsible for side effects."* Basically it outsources the impure operations into separate threads. These threads can be started, paused or cancelled with plain Redux actions from your core application. Thereby threads in Redux Saga make it simple to keep your side-effects away from your core application. However, threads can dispatch actions and have access to the state though.

Redux Saga uses as underlying technology [JavaScript ES6 Generators](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Generator). The code reads like synchronous code. You avoid to have callbacks. The advantage over Redux Thunk is that your actions stay pure and thus can test them well. You will use Redux Saga in a "Hands On" chapter later on.

Apart from Redux Thunk and Redux Sage, there are other side-effect libraries for Redux. If you want to try out observables in JavaScript, you could give [Redux Observable](https://github.com/redux-observable/redux-observable) a shot. It builds up on RxJS, a generic library for reactive programming. If you are interested in another library that uses reactive programming principles too, you can give [Redux Cycles](https://github.com/cyclejs-community/redux-cycles) a try.

In conclusion, as you can see, all these libraries, Redux Saga, Redux Observable and Redux Cycles, make use of different techniques in JavaScript. You can give them a shot to try generators or observables. The whole ecosystem around asynchronous actions is a great playground to try new things in JavaScript.

### Hands On: Todo with Redux Saga

In a previous chapter, you have used Redux Thunk to dispatch asynchronous actions. These were used to add a todo item with a notification whereas the notifaction vanishes after a couple of seconds again. In this chapter you will use Redux Saga instead of Redux Thunk. Therefore, you can install the former library and uninstall the latter one.

{title="Command Line",lang="text"}
~~~~~~~~
npm uninstall --save redux-thunk
npm install --save redux-saga
~~~~~~~~

The action you have used before with a thunk becomes a pure action creator now. It will be used to trigger the saga thread.

{title="Code Playground",lang="javascript"}
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

Now you can introduce your first saga that listens on this particular action, because the action is solely used to trigger the saga thread.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { takeEvery } from 'redux-saga/effects';

...

// sagas

function* watchAddTodoWithNotification() {
  yield takeEvery(TODO_ADD_WITH_NOTIFICATION, ...);
}
~~~~~~~~

Most often you will find one part of the saga watching incoming actions and evaluating them. If an evaluation applies truthfully, it often will call another generator function that handles the side-effect. That way you can keep your side-effect watcher maintainable and don't clutter them with business logic. In the previous example a `takeEvery()` effect of Redux Saga is used to handle every action with the specified action type. Yet there are other effects in Redux Saga such as `takeLatest` which only takes the last of the incoming actions by action type.

{title="Code Playground",lang="javascript"}
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

As you can see, in JavaScript generators you use the `yield` statement to execute asynchronous code synchronously. Only when the function after the `yield` resolves, the code will execute the next line of code. Redux Saga comes with a helper such as `delay()` that can be used to delay the execution by an amount of time. It would be the same as using `setTimeout()` in JavaScript, but the `delay()` helper makes it more concise when using JavaScript generators.

Now, you only have to exchange your middleware in your Redux store from using Redux Thunk to Redux Saga.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import createSagaMiddleware, { delay } from 'redux-saga';
# leanpub-end-insert
import { put, takeEvery } from 'redux-saga/effects';

...

# leanpub-start-insert
import createSagaMiddleware from 'redux-saga';
# leanpub-end-insert

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

That's it. You Todo application should run with Redux Saga instead of Redux Thunk now. The final application can be found in the [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/10.0.0). In the future, it is up to you when using Redux to decide on an asynchronous actions library. Is it Redux Thunk or Redux Saga? Or do you decide to try something new with Redux Observable or Redux Cycles? The Redux ecosystem is full of seducement to try cutting edge JavaScript features such as generators or observables.
