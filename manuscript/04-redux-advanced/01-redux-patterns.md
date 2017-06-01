# Redux Patterns and Best Practices

There are several patterns and best practices that you can apply in a Redux application. I will go through a handful of them to point you in the right direction. However, the evolving patterns and best practices around the ecosystem are fast paced. You want to read more about these topics on your own.

## Using JavaScript ES6

So far, you have written your Redux code mostly in JavaScript ES5. Redux is inspired by the functional programming paradigm and uses a lot of its concepts: immutable data structures and pure functions. When using Redux in your scaling application, you will find yourself often using pure functions that solve only one problem. For instance, a action creator only returns an action, a reducer only returns the new state and a selector only returns derived properties. You will embrace this mental model and use it in Redux agnostic code too. You will see yourself more often using functions that only solve one problem, using higher order functions to return reusable functions and compose functions into each other. You will move toward a functional programming style with Redux.

JavaScript ES6 and beyond complements a functional programming style in JavaScript perfectly. You only have to look at the following example to understand how much more concise higher order functions can be written with JavaScript ES6 arrow functions.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// JavaScript ES5
function higherOrderFunction() {
  return function () {
      ...
  };
}

// JavaScript ES6
const higherOrderFunction = () => { ... };
~~~~~~~~

It's a higher order function that is much more readable in JavaScript ES6. You will find yourself more often using higher order functions when programming in a functional style. It will happen that you not only use one higher order function, but a higher order function that returns a higher order function that returns a function. Again it becomes easier to read when using JavaScript ES6.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// JavaScript ES5
function higherOrderFunction() {
  return function () {
      return function () {
        ...
      }
  };
}

// JavaScript ES6
const higherOrderFunction = () => () => { ... };
~~~~~~~~

I encourage you to apply these in your Redux code.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// action creator
const doAddTodo = (id, name) => ({
  type: ADD_TODO,
  todo: { id, name },
});

// selector
const getTodos = (state) => state.todos;

// action creators using Redux Thunk
const showNotificationWithDelay = (text) => (dispatch) => {
  dispatch(doShowNotification(text));
  setTimeout(() => dispatch(doHideNotification()), 1000);
}
~~~~~~~~

The JavaScript community shifts in the direction of functional programming and embraces more than ever this style. Functional programming let's you write more predictable code by embracing pure functions without side-effects and immutable data structures. JavaScript ES6 makes it easier and more readable to write in a functional style.

## Naming Conventions

In Redux you have a handful of different types of functions. You have action creators, selectors and reducers. It is always good to name them accordingly to their type. Other developers will have an easier time identifiying the function type. Just following a naming convention for your functions, you can give yourself and others a valuable information about the function itself.

Personally I follow this naming convention with Redux functions. It uses prefixes for each function type:

* action creators: **do**Something
* reducers: **apply**Something
* selectors: **get**Something

In the previous chapters, the example code always used this naming convention. In addition, it clearifies things when using higher order functions. Remember the `mapDispatchToProps()` function when connecting Redux to React?

{title="Code Playground",lang="javascript"}
~~~~~~~~
const mapStateToProps = (state) => ({
  todos: getTodos(state),
});

const mapDispatchToProps = (dispatch) => ({
   onToggleTodo: (id) => dispatch(doToggleTodo(id)),
});
~~~~~~~~

The functions themselves become more concise by using JavaScript ES6 arrow functions. But there is another clue that makes proper naming so powerful. Solely from a naming perspective, you can see that the `mapStateToProps()` and `mapDispatchToProps()` functions transform the Redux world to another world. The connected component doesn't know about selectors or actions creators. As you can see, that is already expressed in the transformed props and functions. THey are named `todos` and `onToggleTodo`. There are no remains from the Redux world, from a functional perspective but also from a pure naming perspective. That's powerful, because your underlying components are Redux agnostic.

So far, the topic was only about function naming. But there is another part in Redux that can be named properly: action types. Consider the following action type names:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const ADD_TODO = 'ADD_TODO';
const TODO_ADD = 'TODO_ADD';
~~~~~~~~

Most cultures read from left to right. That's conveyed in programming too. So which action type naming makes more sense? It is the verb or the subject? You can decide on your own, but become clear about a consistent naming convention for your action types. Personally, I find it easier to scan when I have the subject first. When using Redux Logger in a scaling application where a handful actions can be dispatched at once, I find it easier to scan by subject than by verb.

You can even go one step further and apply the subject as domain prefix.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const todo/ADD = 'todo/ADD';
~~~~~~~~

These are only personal naming conventions for those types of functions and constants in Redux. You can come up with your own. But do yourself and your fellow developers a favor and reach an agreement first and then apply them consistently through your code base.

## The Relationship between Actions and Reducers

Actions and reducers are not strictly coupled. They only share an action type. A dispacthed action, for example with the action type `SOMETHING_ADD`, can be captured in multiple reducers that utilize `SOMETHING_ADD`. That's an important fact when implementing a scaling state management architecture in your application.

When coming from an object-oriented programming background though, you might abuse actions/reducers as setters and selectors as getters. You will couple actions and reducers in a 1:1 relationship. It will call it the **command pattern** in Redux. It can be useful in some scenarios, as I will point out later, but in general it's not the philosophy of Redux.

Redux can be seen as event bus of your application. You can send events (actions) with a payload and an identifier (action type) into the bus and it will pass potential consumer (reducers). A part of these consumers is interested in the event. That's what I call the **event pattern** that Redux embraces.

You can say that the higher you place your actions on the spectrum of abstraction, the more reducers are interested in it. The action becomes an event. The lower you place your actions on the spectrum of abstraction, most often only one reducer can consume it. The action becomes a command. It is a concrete action rather than an abstract action. It is important to note though that you have to keep the balance between abstraction and concreteness. Too abstract actions can lead to a mess when too many reducers consume it. Too concrete actions might be only used by one reducer all the time. Most developers run into the latter scenario though. In Redux, obviously depending on your application, it should be a healthy mix of both.

In the book you have encountered most of the time a relationship of 1:1 between action and reducer. Let's take an action that completes a todo as demonstration:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doCompleteTodo(id) {
  return {
    type: COMPLETE_TODO,
    todo: { id },
  };
}

function todosReducer(state = [], action) {
  switch(action.type) {
    case COMPLETE_TODO : {
      return applyCompleteTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

Now imagine that there should be a measuring of the progress of the Todo application user. The progress will always start at zero when the user opens the application. When a todo gets completed, the progress should increase by one. A potential easy solution could be counting all completed todo items. However, since there could be completed todo items already, and you want to measure the completed todo items in this session, the solution would not suffice. The solution could be a second reducer that counts the completed todos in this session.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function progressReducer(state = 0, action) {
  switch(action.type) {
    case COMPLETE_TODO : {
      return state++;
    }
    default : return state;
  }
}
~~~~~~~~

The counter will increment when a todo got completed. Now you can easily measure the progress of the user. Suddenly, you have a 1:2 relationship between action and reducer. Nobody forces you not to couple action and reducer in a 1:1 relationship, but it always makes sense to be creative in this manner. What would happen otherwise? Regarding the progress measurement issue, you might would come up with a second action type and couple it to the previous reducer:

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function doTrackProgress() {
  return {
    type: PROGRESS_TRACK,
  };
}
# leanpub-end-insert

function progressReducer(state = 0, action) {
  switch(action.type) {
# leanpub-start-insert
    case PROGRESS_TRACK : {
# leanpub-end-insert
      return state++;
    }
    default : return state;
  }
}
~~~~~~~~

The action would be dispatched in paralell with the `COMPLETE_TODO` action.

{title="Code Playground",lang="javascript"}
~~~~~~~~
dispatch(doCompleteTodo('0'));
dispatch(doTrackProgress());
~~~~~~~~

But that would miss the point in Redux. You would want to come up with these commonalities to make your actions more abstract and be used by multiple reducers. My rule of thumb for this topic: Approach your actions as concrete actions with a 1:1 relationship to their reducers, but keep yourself always open to reuse them as more abstract actions in other reducers.

## State Keys

- makes more sense when command pattern

## Folder Organization

- technical, feature
- top level when event pattern, feature level when command pattern

### Ducks

- as long as action and reducer are coupled it makes sense
- but they shouldn't be coupled. command pattern should be avoided and state keys are only useful for certian scenarios
