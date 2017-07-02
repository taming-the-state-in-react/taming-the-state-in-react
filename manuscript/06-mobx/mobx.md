# MobX

[MobX](https://mobx.js.org/) advertises itself as simple yet scalable state management library. It was created and introduced by [Michel Weststrate](https://twitter.com/mweststrate) and heavily used, thus battle tested, in his own company.

MobX is an alternative to Redux for state management. It grows in popularity even though only a fraction of people uses it as a state managemenr alternative to Redux. In a later chapter, you can read about the differences between both libraries for state management, because you may want to make an informed decision on whether you should use Redux or MobX to scale your state management.

The book will showcase MobX as alternative to Redux. However, it will not allocate the same space than Redux and thus not deeply dive into the topic. Because MobX doesn't follow an opinionated way of how to structure your state management, it is difficult to tackle it from all angles. There are several ways on where to put your state and how to update it. The book shows only a few opionated ways, but doesn't showcase all of them.

The library uses heaviliy JavaScript decorators that are not widely adopted and supported by browsers yet. But they are not mandatory and you can avoid using them with plain functions. You can find these plain functions in the [official documentation](https://mobx.js.org/). However, this book will showcase the usage of these decorators, because it is another exciting way of using JavaScript.

Along the way of the following chapters you can decide to opt-in any time the [MobX + React DevTools](https://github.com/mobxjs/mobx-react-devtools). You can install the node package with npm and place the `DevTools` component that comes from the library somewhere between your components.

## Introduction

MobX is often used in applications that have a view layer such as React. Thus the state, similiar to Redux, needs to be connected to the view. It needs to be connected in a way that the state can be updated and the updated state flows back into the view.

{title="Code Playground",lang="javascript"}
~~~~~~~~
View -> MobX -> View
~~~~~~~~

The schema can be elaborated to give more detail about MobX and its parts.

{title="Code Playground",lang="javascript"}
~~~~~~~~
View -> (Actions) -> State -> (Computed Values) -> Reactions -> View
~~~~~~~~

It doesn't need to be necessarily the view layer, but when using MobX in an application with components, most likely the view will either mutate the state directly or use a MobX action to mutate it. It can be as simple as a `onClick` handler in a component that triggers the mutation. However, the mutation could also be triggered by a side-effect (e.g. scheduled event).

The state in MobX isn't immutable, thus you can mutate the state directly. Actions in MobX can be used to mutate the state too, but they are not mandatory. You are allowed to mutate the state directly. There is no opinionated way around how to update the state. You have to come up with your own best practice.

In MobX the state becomes observable. Thus, when the state changes, the application reacts to the changes with so called reactions. The part of your application that uses these reactions becomes reactive. For instance, a MobX reaction can be as simple as a view layer update. The view layer in MobX becomes reactive by using reactions. It will update when the state in MobX updates.

In between of an observable MobX state and MobX reactions are computed values. These are not mandatory, similar to the MobX Actions, but add another fine-grained layer into your state. Computed values are derived properties from the state or from other computed values. Apart from the MobX state itself, they are consumed by reactions too. When using computed values, you can keep the state itself in a simple structure. Yet you can derive more complex properties from the state with computed values that are used in reactions too. Computed values evaluate lazily when used in reactions when the state changes. They don't necessarily update every time the state changes, but only when they are consumed in a reaction that updates the view.

These are basically all parts in MobX. The state can be mutated directly or by using a MobX action. Reactions observe these state changes and consume the state itself or computed values from the state or other computed values. Both ends, actions and reactions, can simply be connected to a view layer such as React. The connection can happen in a straight forward way or with a bridging library as you will experience it in the following chapters.

### Observable State

The state in MobX can be everything from JavaScript primitives to complex objects, arrays or only references over to classes that encapsulate the state. Any of these properties can be made `observable` by MobX. When the state changes, all the reactions, for instance the reaction of the view layer, will run to re-render the view. State in MobX isn't managed in one global state object. It is managed in multiple states that are most of the time called stores or states.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const { observable } = mobx;

class TodoStore {
  @observable todos = [];
}

const todoStore = new TodoStore();
~~~~~~~~

Keep in mind that it doesn't need to be managed in a store instance that comes from a JavaScript class. It can be a plain list of todos. The way of using stores to manage your MobX state is already opinionated. Since there are a couple of different ways on where to store your state in MobX, the book will teach the straight forward way of mananging it in stores. In the end, stores enable you to manage a predictable state where every store can be kept responsible for its own substate.

The state in MobX can be mutated directly without actions:

{title="Code Playground",lang="javascript"}
~~~~~~~~
todoStore.todos.push({ id: '0', name: 'learn redux', completed: true });
todoStore.todos.push({ id: '0', name: 'learn mobx', completed: false });
~~~~~~~~

That means as well, that the store instances can leak into the view layer and an `onClick` handler could mutate the state directly in the store.

State and view layer can be coupled very closely in MobX. In comparison to Redux, it doesn't need to use explicit actions to update the state indirectly. You will get to know more about MobX actions in a later chapter.

### Autorun

The autorun functionality in MobX is not often seen. It is similar to the `subscription()` method of the Redux store. It is always called when the observable state in MobX changes and once in the beginning when the MobX state initializes. Similiar to the `subscription()` method of the Redux store, it is later on used to make the view layer reactive with MobX. The `autorun` function is only one way to produce a reaction on MobX.

However, you can use it to experiment with your state updates while learning MobX. You can simply add it to your `TodoStore` example.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const { observable, autorun } = mobx;
# leanpub-end-insert

class TodoStore {
  @observable todos = [];
}

const todoStore = new TodoStore();

# leanpub-start-insert
autorun(() => console.log(todoStore.todos.length));
# leanpub-end-insert

todoStore.todos.push({ id: '0', name: 'learn redux', completed: true });
todoStore.todos.push({ id: '0', name: 'learn mobx', completed: false });
~~~~~~~~

It will run the first time when the state initializes, but then every time again when the observable state updates. You can open the [MobX Playground](https://jsbin.com/vawugugugu/4/edit?js,console) to experiment with it.

### Actions

As mentioned, the state can be mutated directly in MobX. But a handful of people would argue that it is a bad practice. It couples the state mutation to close to the view layer when you start to mutate the state directly in an `onClick` handler. Therefore, you can use MobX actions to decouple the state update and keep your state updates at one place.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const { observable, autorun, action } = mobx;
# leanpub-end-insert

class TodoStore {
  @observable todos = [];

# leanpub-start-insert
  @action addTodo(todo) {
    this.todos.push(todo);
  }
# leanpub-end-insert
}

const todoStore = new TodoStore();

autorun(() => console.log(todoStore.todos.length));

todoStore.addTodo({ id: '0', name: 'learn redux', completed: true });
todoStore.addTodo({ id: '1', name: 'learn mobx', completed: false });
~~~~~~~~

However, MobX is not opionated about the way you update your state. You can use actions or mutate the state directly without an action.

{title="Code Playground",lang="javascript"}
~~~~~~~~
class TodoStore {
  @observable todos = [];

# leanpub-start-insert
  addTodo(todo) {
# leanpub-end-insert
    this.todos.push(todo);
  }
}
~~~~~~~~

In order to enforce state updates with actions, you can opt-in the `useStrict()` functionality by MobX. After you have set it to true, every state update needs to happen via an action. This way you enforce the decoupling of state mutation and view with actions.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const { observable, autorun, action, useStrict } = mobx;
# leanpub-end-insert

# leanpub-start-insert
useStrict(true);
# leanpub-end-insert

class TodoStore {
  @observable todos = [];

# leanpub-start-insert
  @action addTodo(todo) {
# leanpub-end-insert
    this.todos.push(todo);
  }
}

const todoStore = new TodoStore();

autorun(() => console.log(todoStore.todos.length));

todoStore.addTodo({ id: '0', name: 'learn redux', completed: true });
todoStore.addTodo({ id: '1', name: 'learn mobx', completed: false });
~~~~~~~~

You can test the MobX action and the `useStrict()` function in the [MobX Playground](https://jsbin.com/qazazajusa/1/edit?js,console).

### Computed Values

Computed values are derived properties from the state or other computed values. They have no side-effects and thus are pure functions. The computed values help you to keep your state structure simple yet can derive complex properties from it. For instance, when you would filter a list of todos for their `completed` property, you could compute the values of incompleted todo items.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const { observable, action, computed } = mobx;

class TodoStore {
  @observable todos = [];

  @action addTodo(todo) {
    this.todos.push(todo);
  }

# leanpub-start-insert
  @computed get incompleteTodos() {
    return this.todos.filter(todo => !todo.completed);
  }
# leanpub-end-insert
}
~~~~~~~~

The computation happens reactively when the state has changed and a reaction asks for it. Thus, these computed values are at your disposal, apart from the state itself, for your view layer later on. In addition, they don't compute actively every time but rather only compute reactively when a reaction demands it. You can experiment with it in the [MobX Playground](https://jsbin.com/didenujipi/3/edit?js,console).

# MobX in React

The basics in MobX should be clear by now. The state in MobX is mutable and can be mutated directly, by actions too or only by actions, and not directly, when using the strict mode. When scaling your state management in MobX, you would keep it in multiple yet manageable stores to keep it maintainable. These stores can expose actions and computed values, but most important they make their properties observable. All of these facts already give you an opionated way of how to store state (e.g. with stores) and how to update the state (e.g. explicit actions with strict mode). However, you can decide to use a different opionated approach.

Now, every time an observable property in a store changes, the `autorun` function of MobX runs. The `autorun` makes it possible to bridge the MobX state updates over to other environments. For instance, it can be used in a view layer, such as React, to re-render it every time the state changes. MobX and React match very well. Both libraries solve their own problem, but can be used together to build a sophisticated scaling application. The React view layer can receive the state from MobX, but also can mutate the state. It can connect to both ends: getting state and mutating it.

When you start to introduce React to your MobX Playground, you could begin to display the list of todo items from your `todoStore`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
class TodoList extends React.Component {
  render() {
    return (
      <div>
        {this.props.todoStore.todos.map(todo =>
          <div key={todo.id}>
            {todo.name}
          </div>
        )}
      </div>
    );
  }
}

ReactDOM.render(
  <TodoList todoStore={todoStore} />,
  document.getElementById('app')
);
~~~~~~~~

However, when you update your MobX store nothing happens. The view layer is not notified about any state updates, because these happen outside of React. You can use the `autorun` function of MobX to introduce a naive re-rendering of the view layer.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function render() {
# leanpub-end-insert
  ReactDOM.render(
    <TodoList todoStore={todoStore} />,
    document.getElementById('app')
  );
# leanpub-start-insert
}
# leanpub-end-insert

# leanpub-start-insert
autorun(render);
# leanpub-end-insert
~~~~~~~~

Now you have one initial rendering of the view layer, because the `autorun` is running once initially, and successive renderings when the MobX state updates. You can play around with it in the [MobX Playground](https://jsbin.com/vobicoqifu/1/edit?html,js,output).

There exists a neat library that bridges from MobX to React: [mobx-react](https://github.com/mobxjs/mobx-react). It spares you to use the `autorun` reaction in order to re-render the view layer. Instead it uses a `observer` decorator, that uses the `autorun` function under the hood, to produce a reaction. The reaction simply flushes the update to a React component to re-render it. It makes your React view layer reactive and re-renders it when the observable state in MobX has changed.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const { observer } = mobxReact;
# leanpub-end-insert

...

# leanpub-start-insert
@observer
# leanpub-start-insert
class TodoList extends React.Component {
  render() {
    return (
      <div>
        {this.props.todoStore.todos.map(todo =>
          <div key={todo.id}>
            {todo.name}
          </div>
        )}
      </div>
    );
  }
}

ReactDOM.render(
  <TodoList todoStore={todoStore} />,
  document.getElementById('app')
);

...
~~~~~~~~

Again you can play around with it in the [MobX Playground](https://jsbin.com/foqonacowi/1/edit?html,js,output). If you want to use the `TodoList` component as functional component, you can use the `observer` as well.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const TodoList = observer(function (props) {
  return (
    <div>
      {props.todoStore.todos.map(todo =>
        <div key={todo.id}>
          {todo.name}
        </div>
      )}
    </div>
  );
});
~~~~~~~~

You can find the example in the following [MobX Playground](https://jsbin.com/qucesewiwe/1/edit?html,js,output).

## Local State

As mentioned before, MobX is not opinionated about how to store your state and how to update it. It goes so far, that you can even exchange your local state in React, `this.state` and `this.setState()`, with MobX. The case can be demonstrated by introducing a component that adds a todo item to the list of todo items from the previous example.

{title="Code Playground",lang="javascript"}
~~~~~~~~
ReactDOM.render(
# leanpub-start-insert
  <div>
    <TodoAdd todoStore={todoStore} />
# leanpub-end-insert
    <TodoList todoStore={todoStore} />
# leanpub-start-insert
  </div>,
# leanpub-end-insert
  document.getElementById('app')
);
~~~~~~~~

The `TodoAdd` component only renders an input field to capture the name of the todo item and a button to create the todo item.

{title="Code Playground",lang="javascript"}
~~~~~~~~
@observer
class TodoAdd extends React.Component {
  render() {
    return (
      <div>
        <input
          type="text"
          value={this.input}
          onChange={this.onChange}
        />
        <button
          type="button"
          onClick={this.onSubmit}
        >Add Todo</button>
      </div>
    );
  }
}
~~~~~~~~

The two handlers class methods are missing. The `onChange` handler can be an action itself to update a internally managed value of the input field.

{title="Code Playground",lang="javascript"}
~~~~~~~~
@observer
class TodoAdd extends React.Component {

# leanpub-start-insert
  @observable input = '';

  @action onChange = (event) => {
    this.input = event.target.value;
  }
# leanpub-end-insert

  render() {
    ...
  }
}
~~~~~~~~

This way, the `input` property is not allocated in the local state of React, but in the observable state of MobX. The `observer` decorator makes sure that the component stays reactive to its observed properties. The `onSubmit` handler finally creates the todo item yet alters the local state of the component again, because it has to reset the input value and increments the identifier.

{title="Code Playground",lang="javascript"}
~~~~~~~~
@observer
class TodoAdd extends React.Component {

  @observable input = '';
# leanpub-start-insert
  @observable id = 0;

  @action onSubmit = () => {
    this.props.todoStore.addTodo({
      id: this.id,
      name: this.input,
      completed: false,
    });

    this.id++;
    this.input = '';
  }
# leanpub-end-insert

  @action onChange = (event) => {
    ...
  }

  render() {
    ...
  }
}
~~~~~~~~

MobX is able to take over the local state of React. You wouldn't need to use `this.state` and `this.setState()` anymore. The previous example can be found in the [MobX Playground](https://jsbin.com/sonate/1/edit?html,js,output). Again you experience that MobX isn't opinionated about the way the state is managed. You can have the state management encapsulated in a store class or couple it next to a component as local state. It can make React local state obsolete. But should it? That is your own decision.

## Scaling Reactions

Each component can be decorated with an observer to be reactive to observable state changes in MobX. When introducing a new component to display a todo item, you can decorate it as well. This `TodoItem` component receives the todo property, but also the `todoStore` in order to complete a todo item.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const TodoItem = observer(({ todo, todoStore }) => {
  return (
    <div>
      {todo.name}
      <button
         type="button"
         onClick={() => todoStore.toggleCompleted(todo)}
       >
       {todo.completed
         ? "Incomplete"
         : "Complete"
       }
      </button>
    </div>
  );
});
~~~~~~~~

Notice that the `TodoItem` is a functional stateless component. In addition, in order to complete a todo item, you would have to introduce the `toggleCompleted` action in the `TodoStore`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
class TodoStore {
  @observable todos = [];

  ...

# leanpub-start-insert
   @action toggleCompleted(todo) {
    todo.completed = !todo.completed;
  }
# leanpub-end-insert
}
~~~~~~~~

The `TodoList` component could use the `TodoItem` component now.

{title="Code Playground",lang="javascript"}
~~~~~~~~
@observer
class TodoList extends React.Component {
  render() {
    return (
      <div>
        {this.props.todoStore.todos.map(todo =>
# leanpub-start-insert
          <TodoItem
            todoStore={this.props.todoStore}
            todo={todo}
            key={todo.id}
          />
# leanpub-end-insert
        )}
      </div>
    );
  }
};
~~~~~~~~

In the running application you should be able to complete a todo item. The benefit of splitting up one reactive component into multiple reactive components can be seen when adding two `console.log()` statements.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const TodoItem = observer(({ todo, todoStore }) => {
# leanpub-start-insert
  console.log('TodoItem: ' + todo.name);
# leanpub-end-insert
  return (
    ...
  );
});

@observer
class TodoList extends React.Component {
# leanpub-start-insert
  console.log('TodoList');
# leanpub-end-insert
  render() {
    ...
  }
};
~~~~~~~~

You can open up the application in the [MobX Playground](https://jsbin.com/sonate/6/edit?js,console,output). When you add a todo item, you will get the `console.log()` outputs for the `TodoList` component and only the newly created `TodoItem`. When you complete a todo item, you will only get the `console.log()` of the completing `TodoItem` component. The reactive component only updates when their observable state changes. Everything else doesn't update, because the `observer` decorator implements under the hood the `shouldComponentUpdate()` lifecycle method of React to prevent the component from updating when nothing has changed. You can read more about optimizing MobX performance in React in the [official documentation](https://mobx.js.org/best/react-performance.html).

## Inject Stores

So far, the application passes down the store from the React entry point via props to its child components. They are already passed down more than one layer. However, the store(s) could be used directly in the components by using them directly (when accessible in the file). They are only observable state. Since MobX is not opinionated about where to put state, the observable state, in this case stores, could live anywhere. But as mentioned, the book tries to give an opionated approach as best practice.

The [mobx-react](https://github.com/mobxjs/mobx-react) library provides you with two helpers to pass the observable state implictly down to the components (via React`s context) rather than passing them through every component layer explictly.

The first helper is the `Provider` component that passes down all the neccessary observable states down.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const { observer, Provider } = mobxReact;
# leanpub-end-insert

...
~~~~~~~~

You can use it in the React entry point to wrap your component tree. In addition, you can pass it any observable state that should be passed down. In this case, the observable state is the store instance.

{title="Code Playground",lang="javascript"}
~~~~~~~~
...

ReactDOM.render(
# leanpub-start-insert
  <Provider todoStore={todoStore}>
    <div>
      <TodoAdd />
      <TodoList />
    </div>
  </Provider>,
# leanpub-end-insert
  document.getElementById('app')
);
~~~~~~~~

However, it could be multiple stores or only a couple of observable primitives.

{title="Code Playground",lang="javascript"}
~~~~~~~~
<Provider
  storeOne={storeOne}
  storeTwo={storeTwo}
  anyOtherState={anyOtherState}
>
  ...
</Provider>
~~~~~~~~

The second helper from the library is the `inject` decorator. You can use it for any component down your component tree, that is wrapped somewhere above by the `Provider` component, to retrieve the provided observable state from the React context as props.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const { observer, inject, Provider } = mobxReact;
# leanpub-end-insert

...

# leanpub-start-insert
@inject('todoStore') @observer
# leanpub-end-insert
class TodoAdd extends React.Component {
  ...
}
~~~~~~~~

The `TodoAdd` component already has access to the `todoStore` now. You can add the injection to the other components too. It can be used as function for functional stateless components.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const TodoItem = inject('todoStore')(observer(({
# leanpub-end-insert
  todo, todoStore
}) =>
  <div>
    {todo.name}
    <button
      type="button"
      onClick={() => todoStore.toggleCompleted(todo)}
    >
    {todo.completed
      ? "Incomplete"
      : "Complete"
    }
    </button>
  </div>
# leanpub-start-insert
));
# leanpub-end-insert
~~~~~~~~

The `TodoList` component doesn't need to manually pass down the `todoStore` anymore. The `TodoItem` already accesses it via its `inject` helper.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
@inject('todoStore') @observer
# leanpub-end-insert
class TodoList extends React.Component {
  render() {
    return (
      <div>
        {this.props.todoStore.todos.map(todo =>
# leanpub-start-insert
          <TodoItem
            todo={todo}
            key={todo.id}
          />
# leanpub-end-insert
        )}
      </div>
    );
  }
};
~~~~~~~~

Every component can access the observable state, that is passed to the `Provider` component, with the `inject` decorator. This way you keep a clear separation of state and view layer. You can access the project in the [MobX Playground](https://jsbin.com/sonate/7/edit?js,output) again.

# Advanced MobX

## Reactions

- observer
- autorun
- when https://mobx.js.org/refguide/when.html
- more fine-graned than autorn: https://mobx.js.org/refguide/reaction.html but not into detail here

https://egghead.io/lessons/react-mobx-fundamentals-writing-your-own-reactions-using-when-and-autorun

## Transactions

- https://medium.com/@mweststrate/becoming-fully-reactive-an-in-depth-explanation-of-mobservable-55995262a254

https://mobx.js.org/refguide/transaction.html

## Asynchronous Actions

- https://mobx.js.org/getting-started.html#demo
- https://mobx.js.org/best/actions.html

## State and Stores

- local state, basically operating on primitives that are coupled to components
- doesn't need to be coupled, can be somewhere outside

- objects, arrays, references, class instances

- stores are only one way to manage state in mobx
- https://mobx.js.org/best/store.html
- domain stores (entity state), ui stores (view state)

## State Architecture

- make it opinionated your way!
- defintely align on a state architecture:
- how to manage local state
- use strict?
- inject or passing?
- store insatnces with actions and computations etc?

- https://t.co/0imjjYENUo?ssr=true
- ref as outline mobx-state-tree
- ref Michel video youtube

## Redux Comparison

 - it’s defining power comes from it’s reactive nature.
 - It removes vulnerable manual subscriptions and replaces connects with more individual component observers.
 - So while it creates more “smart” components, at scale it actually preforms better.

- no normalization
- ...
- no strict boundaries between local and global state
- used like two.way data bdingin
- useStrict should be best practice

## Redux to MobX, MobX to Redux

- refacotr
- only bridge changes with container
- no normalization anymore

## Hands On: Snake with MobX

- take local state snake as begin

## Hands On: Todo App with MobX

- or leave it out, or challenge and leave out HN MobX

## Challenge: Hacker News App with MobX
