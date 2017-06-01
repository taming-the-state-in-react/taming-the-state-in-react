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

These are only my personal naming conventions for those types of functions and constants in Redux. You can come up with your own. But do yourself and your fellow developers a favor and reach an agreement first and then apply them consistently through your code base.

## Nested Combined Reducers

https://stackoverflow.com/questions/36786244/nested-redux-reducers

## Command vs. Event Pattern

- https://hackernoon.com/dispatch-redux-actions-as-events-not-commands-4de8a92b1ea5

- single vs multiple reducers
- to close to command should be evaluated as local state
- Action to Reducer like 1:N

Last but not least, I want to give you clarification on the relationship between actions and reducers. You know that your application can have multiple actions and reducers. But how do they relate to each other?

- showcase one action that is captured by multiple reducers
- that will be explained in greater detail in the Command and Event patterns in Redux

## State Keys

- makes more sense when command pattern

## Folder Organization

- technical, feature
- top level when event pattern, feature level when command pattern

### Ducks

- as long as action and reducer are coupled it makes sense
- but they shouldn't be coupled. command pattern should be avoided and state keys are only useful for certian scenarios
