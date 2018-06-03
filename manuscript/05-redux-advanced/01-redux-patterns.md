# Redux Patterns, Techniques and Best Practices

There are several patterns and best practices that you can apply in a Redux application. I will go through a handful of them to point you in the right direction. However, the evolving patterns and best practices around the ecosystem are changing all the time. You will want to read more about these topics on your own.

## Using JavaScript ES6

So far, you have written your Redux code mostly in JavaScript ES5. Redux is inspired by the functional programming paradigm and uses a lot of its concepts: immutable data structures and pure functions. When using Redux in your scaling application, you will often find yourself using pure functions that solve only one problem. For instance, an action creator only returns an action object, a reducer only returns the new state and a selector only returns derived properties. You will embrace this mental model and use it in Redux agnostic code, too. You will see yourself more often using functions that only solve one problem, using higher-order functions to return functions and compose functions into each other. You will move toward a functional programming style with Redux.

JavaScript ES6 and beyond complements the functional programming style in JavaScript perfectly. You only have to look at the following example to understand how much more concisely higher-order functions can be written with JavaScript ES6 arrow functions.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// JavaScript ES5
function higherOrderFunction(foo) {
  return function (bar) {
      ...
  };
}

// JavaScript ES6
const higherOrderFunction = (foo) => (bar) => { ... };
~~~~~~~~

It's a higher-order function that is much more readable in JavaScript ES6. You will find yourself using higher-order functions more often when programming in a functional style. It will happen that you not only use one higher-order function, but a higher-order function that returns a higher-order function that returns a function. Again it becomes easier to read when using JavaScript ES6.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// JavaScript ES5
function higherOrderFunction(foo) {
  return function (bar) {
      return function (qwe) {
        ...
      }
  };
}

// JavaScript ES6
const higherOrderFunction = (foo) => (bar) => (qwe) => { ... };
~~~~~~~~

I encourage you to apply these in your Redux code. It would turn out the following way:

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

The JavaScript community made a great shift toward functional programming with the rising popularity of React and Redux. Functional programming let's you write more predictable code by embracing pure functions, raising awareness about side-effects, and keeping data structures immutable. JavaScript ES6 and beyond make it easier and more readable to write in a functional style.

## Naming Conventions

In Redux, you have a handful of different types of functions. You have action creators, selectors and reducers. It is always good to name them accordingly to their type. Other developers will have an easier time identifying the function type. Just following a naming convention for your functions, you can give yourself and others valuable information about the function itself.

Personally, I follow this naming convention with Redux functions. It uses prefixes for each function type:

* action creators: **do**Something
* reducers: **apply**Something
* selectors: **get**Something
* sagas: **watch**Something, **handle**Something

In the previous chapters, the example code always used this naming convention. In addition, it clarifies things when using higher-order functions. Remember the `mapDispatchToProps()` function when connecting Redux to React?

{title="Code Playground",lang="javascript"}
~~~~~~~~
const mapStateToProps = (state) => ({
  todos: getTodos(state),
});

const mapDispatchToProps = (dispatch) => ({
   onToggleTodo: (id) => dispatch(doToggleTodo(id)),
});
~~~~~~~~

The functions themselves become more concise by using JavaScript ES6 arrow functions. But there is another clue that makes proper naming so powerful. Solely from a naming perspective, you can see that the `mapStateToProps()` and `mapDispatchToProps()` functions transform the returned properties from the Redux world to the React world. The connected component doesn't know about selectors or actions creators anymore. As you can see, that is already expressed in the transformed props and functions. They are named `todos` and `onToggleTodo`. There are no remains from the Redux world, from a technical perspective but also from a pure naming perspective. That's powerful, because your underlying components are Redux agnostic.

So far, the chapter was only about the naming of functions in the Redux world. But there is another part in Redux that can be named properly: action types. Consider the following action type names:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const ADD_TODO = 'ADD_TODO';
const TODO_ADD = 'TODO_ADD';
~~~~~~~~

Most cultures read from left to right. That's conveyed in programming, too. So which action type naming makes more sense? Is it the verb or the subject? You can decide on your own, but become clear about a consistent naming convention for your action types. Personally, I find it easier to scan when I have the subject, the domain in this case, first. When using Redux Logger in a scaling application where a handful actions can be dispatched at once, I find it easier to scan by subject than by verb. You can even go one step further and apply the subject as domain prefix for your action types:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const todo/ADD = 'todo/ADD';
~~~~~~~~

These are only opinionated naming conventions for these types of functions and constants (action types) in Redux. You can come up with your own. But do yourself and your fellow developers a favor and reach an agreement first and then apply them consistently through your code base.

## The Relationship between Actions and Reducers

Actions and reducers are not strictly coupled. They only share an action type. A dispatched action, for example with the action type `TODO_ADD`, can be captured in multiple reducers that utilize `TODO_ADD`. You have done it before in the `todoReducer` and `notificationReducer`. That's an important fact when implementing a larger state management architecture in your application.

When coming from an object-oriented programming background though, you might abuse actions/reducers as setters and selectors as getters. You will couple actions and reducers in a 1:1 relationship. I will call it the **command pattern** in Redux. It can be useful in some scenarios, as I will point out later, but in general it's not the philosophy of Redux.

Redux can be seen as event bus of your application. You can send events (actions) with a payload and an identifier (action type) into the bus and it will pass potential consumer (reducers). Only a part of these consumers is interested in the event. That's what I call the **event pattern** that Redux embraces.

You can say that the higher you place your actions on the spectrum of abstraction, the more reducers are interested in it. The action becomes an event. The lower you place your actions on the spectrum of abstraction, the less reducers are interested in it. In the end, most often only one reducer can consume it when it is placed on a lower place of the spectrum of abstraction. The action becomes a command. It is a concrete action rather than an abstract action. It is important to note, though, that you have to keep the balance between abstraction and concreteness. Too abstract actions can lead to a mess when too many reducers consume it. Too concrete actions might only be used by one reducer all the time. Most developers run into the latter scenario though. In Redux, obviously depending on your application, it should be a healthy balance of both.

In the book, you have encountered a relationship of 1:1 between action and reducer most of the time (except for the `TODO_ADD` action). Let's take an action that completes a todo item as demonstration:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doCompleteTodo(id) {
  return {
    type: TODO_COMPLETE,
    todo: { id },
  };
}

function todosReducer(state = [], action) {
  switch(action.type) {
    case TODO_COMPLETE : {
      return applyCompleteTodo(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

Now, imagine that there should be a measuring of the progress of the Todo application user. The progress will always start at zero when the user opens the application. When a todo gets completed, the progress should increase by one. A potentially easy solution could be counting all completed todo items. However, since there could be completed todo items already, and you want to measure the completed todo items in this session, the solution would not suffice. The solution could be a second reducer that counts the completed todos in this session.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function progressReducer(state = 0, action) {
  switch(action.type) {
    case TODO_COMPLETE : {
      return state++;
    }
    default : return state;
  }
}
~~~~~~~~

The counter will increment when a todo got completed. Now, you can easily measure the progress of your users when they complete todo items. Suddenly, you have a 1:2 relationship between action and reducer. The action that is used for completing a todo item is consumed by two reducers. Nobody forces you not to couple action and reducer in a 1:1 relationship, but it always makes sense to be creative in this manner. What would happen otherwise? Regarding the progress measurement issue, you might have to come up with a second action type and couple it to the previous reducer:

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

The action would be dispatched in parallel with the `COMPLETE_TODO` action.

{title="Code Playground",lang="javascript"}
~~~~~~~~
dispatch(doCompleteTodo('0'));
dispatch(doTrackProgress());
~~~~~~~~

But that would miss the point in Redux. You would want to come up with these commonalities to make your actions more abstract and be used by multiple reducers. My rule of thumb for this: Approach your actions as concrete actions with a 1:1 relationship to their reducers, but keep yourself always open to reuse them as more abstract actions in other reducers.

## Folder Organization

Eventually, your Redux application grows and you cannot manage everything - reducers, action creators, selectors, store and view - in one file. You will have to split up the files. Fortunately, JavaScript ES6 brings [import](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/import) and [export](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/export) statements to distribute functionalities in files. If you are not familiar with these, you should read about them.

In this chapter, I want to show you two approaches to organize your folder and files in a Redux application. The first approach, the **technical folder organization**, is used in smaller applications. Once your application scales and more than one team in your organization is working on the project, you can consider the **feature folder organization**. In addition, you will learn about best practices for your file and folder structure in this chapter.

### Technical Folder Organization

The technical separation of concerns is used in smaller applications. Basically, in my opinion, there are two requirements to use this approach:

* the application is managed by only one person or one team, thus has less conflict potential when working on the same code base
* the application is small from a lines of code perspective and *can* be managed by one person or one team

In conclusion, it depends on the size of the team and the size of the code base. Now, how to separate the files? They get separated by their technical aspects:

{title="Folder Organization",lang="text"}
~~~~~~~~
-app
--reducers
--actions creators
--selectors
--store
--constants
--components
~~~~~~~~

The reducers, action creators, selectors and store should be clear. In these folders you have all the different aspects of Redux. In the components folder you have your view layer. When using React, that would be the place where you will find your React components. In the constants folder you can have any constants, but also the action types of Redux. These can be imported in the action creators and reducers. An elaborated folder/file organization split by technical aspects might look like the following:

{title="Folder Organization",lang="text"}
~~~~~~~~
-app
--reducers
---todoReducer.js
---filterReducer.js
---notificationReducer.js
--actions creators
---filters.js
---todos.js
---notifications.js
--selectors
---filters.js
---todos.js
---notifications.js
--store
---store.js
--constants
---actionTypes.js
--components
---TodoList.js
---TodoItem.js
---Filter.js
---Notifications.js
~~~~~~~~

What are the advantages and disadvantages of this approach? The most important advantage is that reducers and action creators are not coupled. They are loosely arranged in their folders. It embraces the notion of Redux to capture any action in any reducer. Reducers and action creators are not in a 1:1 relationship. In addition, all Redux functionalities are reachable from a top level. None of these functionalities are hidden in a lower level and thus less accessible. This approach embraces the event pattern which was mentioned before. A disadvantage of this approach, hence the two requirements, is that it doesn't scale well. Each technical folder will grow endlessly. There are no constraints except for the separation by type. It can become messy after you have introduced several reducers, action creators and selectors.

### Feature Folder Organization

The second approach, the separation by feature, is most often used in larger applications. You have a greater flexibility in grouping the features, because you can always split up bigger features to smaller ones and thus keep the folders lightweight.

{title="Folder Organization",lang="text"}
~~~~~~~~
-app
--todo
--filter
--notification
--store
~~~~~~~~

An elaborated folder/file organization might look like the following:

{title="Folder Organization",lang="text"}
~~~~~~~~
-app
--todo
---TodoList.js
---TodoItem.js
---reducer.js
---actionCreators.js
---selectors.js
--filter
---Filter.js
---reducer.js
---actionCreators.js
---selectors.js
--notification
---Notifications.js
---reducer.js
---actionCreators.js
---selectors.js
--store
---store.js
~~~~~~~~

This approach, separating by features, is way more flexible than the previous approach. It gives you more freedom to arrange your folders and files as features. When using this approach, there are more ways to accomplish it. You don't necessarily have to follow the example above.

What are the advantages and disadvantages of this approach? It has the same advantages and disadvantages as the technical folder organization but negated. Instead of making action creators and reducers accessible on a top level, they are hidden in a feature folder. In a scaling application with multiple teams, other teams will most likely not reuse your action creators and reducers but implement their own. Another disadvantage is that it groups action creators and reducers in a 1:1 relationship which goes against the overarching idea of Redux. You embrace a command pattern instead of an event pattern. The advantage on the other side, and that's why most teams in a scaling application are using this approach, is that it grows well. Teams can work on separate feature folders and don't run into conflicts. Still, they can follow the overarching state management flow, when using a middleware library like redux-logger.

Even though the feature folder organization bears a lot of pitfalls by embracing the command pattern which was mentioned earlier, it is often used in scaling applications with several development teams. Therefore, I can give one crucial advice to this approach: Try to make your action creators, reducers and selectors accessible to everyone so that they can be reused. It can happen by documentation, word of mouth or another variation of folder/file organization.

### Ducks

In Redux, there exists another concept called ducks. It relates to the organization of action creators, action types and reducers as tuples. The ducks concept bundles these tuples into self contained modules. Often, these modules end up being only one file. The official ducks pattern has a bunch of guidelines which you can read up in the [GitHub repository](https://github.com/erikras/ducks-modular-redux). However, you wouldn't need to apply all of these. For instance, in the Todo application a duck file for the filter domain might look like the following:

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

function doSetFilter(filter) {
  return {
    type: FILTER_SET,
    filter,
  };
}

export default filterReducer;

export {
  doSetFilter,
};
~~~~~~~~

The same file structure would apply for the notification feature and the todo feature itself. The drawbacks of the ducks concept are similar to the feature folder approach. You couple actions and reducers, hence no one will embrace to capture actions in multiple reducers. As long as action and reducer are coupled, the ducks concept makes sense. Otherwise, it shouldn't be applied too often. Instead, you should embrace the idea of Redux to keep your reducers and action creators accessible on a top level.

## Testing

The book will not dive deeply into the topic of testing, but it shouldn't be unmentioned. Testing your code in programming is essential and should be seen as mandatory. You want to keep the quality of your code high and an assurance that everything works. However, testing your code can often be tedious. You have to setup, mock or spy things before you can finally start to test it. Or you you have to cover a ton of edge cases in your one huge code block. But I can give you comfort by saying that this is not the case when testing state management done with Redux. I will show you how you can easily test the necessary parts, keep your efforts low and stay lazy.

Perhaps you have heard about the testing pyramid. There are end-to-end tests, integration tests and unit tests. If you are not familiar with those, the book gives you a quick and basic overview. A unit test is used to test an isolated and small block of code. It can be a single function that is tested by an unit test. However, sometimes the units work well in isolation yet don't work in combination with other units. They need to be tested as a group as units. That's where integration tests can help out by covering whether units work well together. Last but not least, an end-to-end test is the simulation of a real user scenario. It could be an automated setup in a browser simulating the login flow of an user in a web application. While unit tests are fast and easy to write and to maintain, end-to-end tests are the opposite of this spectrum.

How many tests do I need of each type? You want to have many unit tests to cover your isolated functions. After that, you can have several integration tests to cover that the most important functions work in combination as expected. Last but not least, you might want to have only a few end-to-end tests to simulate critical scenarios in your web application. That's it for the general excursion in the world of testing. Now, how does it apply to state management with Redux?

Redux embraces the functional programming style. Your functions are pure and you don't have to worry about any side-effects. A function always returns the same output for the same input. Such functions are easy to test, because you only have to give them an input and expect the output because there is a no side-effect guarantee. That's the perfect fit for unit tests, isn't it? In conclusion, it makes state management testing when build with Redux a pleasure.

In Redux, you have different groups of functions: action creators, reducers, selectors. For each of these groups, you can see a pattern for their input and output. These can be applied to a test pattern which can be used as blueprint for a unit test for each group of functions.

Input Pattern:

* action creators can have an optional input that becomes their optional payload
* selectors can have an optional input that supports them to select the substate
* reducers will always receive a previous state and action

Output Pattern:

* action creators will always return an object with a type and optional payload
* selectors will always return a substate of the state
* reducers will always return a new state

Test Pattern:

* when invoking an action creator, the correct return object should be expected
* when invoking a selector, the correct substate should be expected
* when invoking a reducer, the correct new state should be expected

How does that apply in code? The book will show it in pseudo code, because it will not make any assumption about your testing libraries. Yet it should be sufficient to pick up these patterns for each group of functions (action creators, reducers, selectors) to apply them in your unit tests.

*Action Creators:*

{title="Code Playground",lang="javascript"}
~~~~~~~~
// whereas the payload is optional
const payload = { ... };

const action = doSomething(payload);
const expectedAction = {
  type: 'DO_SOMETHING',
  payload,
};

expect(action).to.equal(expectedAction);
~~~~~~~~

*Selectors:*

{title="Code Playground",lang="javascript"}
~~~~~~~~
// whereas the payload is optional
const state = { ... };
const payload = { ... };

const substate = getSomething(state, payload);
const expectedSubstate = { ... };

expect(substate).to.equal(expectedSubstate)
~~~~~~~~

*Reducers:*

{title="Code Playground",lang="javascript"}
~~~~~~~~
const previousState = { ... };
const action = {
  type: 'DO_SOMETHING',
  payload,
};

const newState = someReducer(previousState, action);
const expectedNewState = { ... };

expect(newState).to.equal(expectedNewState);
~~~~~~~~

These test patterns will always stay the same for their aspects in Redux. You only have to fill in the blanks. You can even give yourself an easier time and setup automated code snippets for your editor of choice. For instance, typing "rts" (abbr. for "redux test selector") gives you the blueprint for a selector test. The other two snippets could be "rtr" (redux test reducer) and "rta" (redux test action). After that, you only have to fill in the remaining things.

These test patterns for state management show you how simple testing becomes when working with the clear constraints of a library like Redux. Everything behaves the same, it is predictable, and thus can be tested every time the in the same way. When setting up automated code snippets, you will save yourself a lot of time, yet have a great test coverage for your whole state management. You can even go one step further and apply [test-driven development (TDD)](https://en.wikipedia.org/wiki/Test-driven_development) which basically means you test before you implement.

There is another neat helper that can ensure that your state stays immutable. Because you never know if you accidentally mutate your state even though it is forbidden in Redux. I guess, there is a handful of libraries around this topic, but I use [deep-freeze](https://github.com/substack/deep-freeze) in my tests to ensure that the state (and even actions) doesn't get mutated.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import deepFreeze from 'deep-freeze';
# leanpub-end-insert

const previousState = { ... };
const action = {
  type: 'DO_SOMETHING',
  payload,
};

# leanpub-start-insert
deepFreeze(previousState);
# leanpub-end-insert

const newState = someReducer(previousState, action);
const expectedNewState = { ... };

expect(newState).to.equal(expectedNewState);
~~~~~~~~

That's it for testing your different aspects when using Redux. It can be accomplished by using unit tests. You could apply integration tests, too, for instance to test an action creator and reducer altogether. After all, you have a blueprint for testing these functions all the time at your hand and there is no excuse anymore to not test your code. If you want to dive into a testing setup for your React application, checkout this [React testing setup and usage walkthrough](https://www.robinwieruch.de/react-testing-tutorial/).

## Error Handling

The topic of error handling is rarely touched in programming. Often, the topic is avoided by the community and it is hard to find a common sense about it. This chapter gives you basic guidance on how you could provide error handling in your Redux application.

Error handling is often involved when making requests to an API. You have learned about asynchronous actions in Redux that can be used for these kind of side-effects. But there was no saying about error handling in those side-effects so far. How to catch the errors and how to make them visible for your application end-user?

Basically, an error in an application can be represented as a state. That's why the topic is discussed in a state management book in the first place. For instance, imagine that you get your todo items from a server request. You would have an API on the server-side that exposes these todo items. Once you fetch these todo items from the API, you would have to deal with error handling, because a request can always fail. The following request returns a JavaScript promise. The fetch can be either successfully resolved in a `then()` method or yields an error in a `catch()` method.

{title="Code Playground",lang="javascript"}
~~~~~~~~
fetch('my/todos/api').then(function(response) {
  return response.json();
}).then(function(todos) {
  // do something with todos
}).catch(function(error) {
  // do something with error
});
~~~~~~~~

When using Redux asynchronous actions with Redux Thunk, the request could live in a thunk:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function getTodos(dispatch) {
  fetch('my/todos/api').then(function(response) {
    return response.json();
  }).then(function(todos) {
    // do something with todos
  }).catch(function(error) {
    // do something with error
  });
}
~~~~~~~~

Now, it would be up to you to store either the todos or the error as state in your Redux store. You could have two potential actions:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const TODOS_FETCH_SUCCESS = 'TODOS_FETCH_SUCCESS';
const TODOS_FETCH_ERROR = 'TODOS_FETCH_ERROR';
~~~~~~~~

These could be used in your Redux Thunk to store both potential outcomes:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function getTodos(dispatch) {
  fetch('my/todos/api').then(function(response) {
    return response.json();
  }).then(function(todos) {
    dispatch({ type: TODOS_FETCH_SUCCESS, todos });
  }).catch(function(error) {
    dispatch({ type: TODOS_FETCH_ERROR, error });
  });
}
~~~~~~~~

The todo reducer would have to deal with both actions now. One that stores the todo items and one that stores the error object.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const initialState = {
  todos: [],
  error: null,
};

function reducer(state = initialState, action) {
  switch(action.type) {
    case 'TODOS_FETCH_SUCCESS' : {
      return applyFetchTodosSuccess(state, action);
    }
    case 'TODOS_FETCH_ERROR' : {
      return applyFetchTodosError(state, action);
    }
    default : return state;
  }
}

...
~~~~~~~~

That's it basically for the state management part. Whereas the `applyFetchTodosError()` function would set the error object in the state, the `applyFetchTodosSuccess()` function would set the list of todos. In addition, the success function would have to reset the error property in the state to null again, because imagine you would do a second request after the first request has failed. When the second request was successful, you would want to store the todo items but reset the error state.

In your view layer, depending on the todo state, you could decide whether to show an error message, because there is an error object in the todo state, or to show the list of todos. When there is an error message displayed, you could provide your end-user with a button to try fetching the todos again. When the second request is successful, the error object is set to null and instead the todo items are set in the state. The view layer could display the list of todo items now.

After all, there is no magic behind error handling in Redux. Whenever an error occurs, you would store it in your state. When the view layer notices an error in the state, it could use conditional rendering to show an error message instead of the assumed result.