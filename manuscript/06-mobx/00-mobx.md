# MobX

The next chapters of the book will dive into an alternative library that provides a state management solution: MobX. However, the book will not allocate the same space as for Redux and thus not deeply dive into the topic. Because MobX doesn't follow an opinionated way of how to structure your state management, it is difficult to tackle it from all angles. There are several ways on where to put your state and how to update it. The book shows only a few opinionated ways, but doesn't showcase all of them.

MobX advertises itself as simple yet scalable state management library. It was created and introduced by [Michel Weststrate](https://twitter.com/mweststrate) and heavily used, thus battle tested, in his own company called Mendix. MobX is an alternative to Redux for state management. It grows in popularity even though only a fraction of people uses it as a state management alternative to Redux. In a later chapter, you can read about the differences between both libraries for state management, because you may want to make an informed decision on whether you should use Redux or MobX to scale your state management.

The library uses heavily JavaScript decorators that are not widely adopted and supported by browsers yet. But they are not mandatory and you can avoid using them with plain functions instead. You can find these plain functions in the [official documentation](https://mobx.js.org/). However, this book will showcase the usage of these decorators, because it is another exciting way of using JavaScript.

Along the way of the following chapters you can decide to opt-in any time the [MobX + React DevTools](https://github.com/mobxjs/mobx-react-devtools). You can install the node package with npm and follow the instructions from the GitHub repository.

## Introduction

MobX is often used in applications that have a view layer such as React. Thus the state, similar to Redux, needs to be connected to the view. It needs to be connected in a way that the state can be updated and the updated state flows back into the view.

{title="Concept Playground",lang="text"}
~~~~~~~~
View -> MobX -> View
~~~~~~~~

The schema can be elaborated to give more detail about MobX and its parts.

{title="Concept Playground",lang="text"}
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

Keep in mind that it doesn't need to be managed in a store instance that comes from a JavaScript class. It can be a plain list of todos. The way of using stores to manage your MobX state is already opinionated. Since there are a couple of different ways on where to store your state in MobX, the book will teach the straight forward way of managing it in stores. In the end, stores enable you to manage a predictable state where every store can be kept responsible for its own substate.

The state in MobX can be mutated directly without actions:

{title="Code Playground",lang="javascript"}
~~~~~~~~
todoStore.todos.push({ id: '0', name: 'learn redux', completed: true });
todoStore.todos.push({ id: '0', name: 'learn mobx', completed: false });
~~~~~~~~

That means as well, that the store instances can leak into the view layer and an `onClick` handler could mutate the state directly in the store.

State and view layer can be coupled very closely in MobX. In comparison to Redux, it doesn't need to use explicit actions to update the state indirectly. You will get to know more about MobX actions in a later chapter.

### Autorun

The autorun functionality in MobX is not often seen. It is similar to the `subscription()` method of the Redux store. It is always called when the observable state in MobX changes and once in the beginning when the MobX state initializes. Similar to the `subscription()` method of the Redux store, it is later on used to make the view layer reactive with MobX. The `autorun` function is only one way to produce a reaction on MobX.

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

It will run the first time when the state initializes, but then every time again when the observable state updates. You can open the [MobX Playground](https://jsbin.com/gezibazuke/1/edit?js,console) to experiment with it.

### Actions

As mentioned, the state can be mutated directly in MobX. But a handful of people would argue that it is a bad practice. It couples the state mutation too close to the view layer when you start to mutate the state directly in a `onClick` handler. Therefore, you can use MobX actions to decouple the state update and keep your state updates at one place.

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

However, MobX is not opinionated about the way you update your state. You can use actions or mutate the state directly without an action.

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

In order to enforce state updates with actions, you can opt-in a configuration with the `configure()` functionality from MobX. There you can pass an object for global MobX confgurations whereas there is one configuration in particular to enforce actions in MobX. After you have set it to true, every state update needs to happen via an action. This way you enforce the decoupling of state mutation and view with actions.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const { observable, autorun, action, configure } = mobx;
# leanpub-end-insert

# leanpub-start-insert
configure({ enforceActions: true });
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

You can test the MobX action and the `configure()` function with the enforced actions in the [MobX Playground](https://jsbin.com/zigapodeke/1/edit?js,console). In addition, it makes always sense to think thoughtfully about your actions. In the previous case, every call of `addTodo()` would lead all relying reactions to run. That's why the autorun function runs every time you add a todo item. So how would you accomplish to add multiple todo items at once without triggering reactions for every todo item? You could have another action that takes an array of todo items.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const { observable, autorun, action } = mobx;

class TodoStore {
  @observable todos = [];

  @action addTodo(todo) {
    this.todos.push(todo);
  }

# leanpub-start-insert
  @action addTodos(todos) {
    todos.forEach(todo => this.addTodo(todo));
  }
# leanpub-end-insert
}

const todoStore = new TodoStore();

autorun(() => console.log(todoStore.todos.length));

# leanpub-start-insert
todoStore.addTodos([
  { id: '0', name: 'learn redux', completed: true },
  { id: '1', name: 'learn mobx', completed: false },
]);
# leanpub-end-insert
~~~~~~~~

That way, the relying reactions only evaluate once after the action got called. You can find the necessary code to play around with in the [MobX Playground](https://jsbin.com/yunebovose/1/edit?js,console).

### Computed Values

Computed values are derived properties from the state or other computed values. They have no side-effects and thus are pure functions. The computed values help you to keep your state structure simple yet can derive complex properties from it. For instance, when you would filter a list of todos for their `completed` property, you could compute the values of uncompleted todo items.

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

The computation happens reactively when the state has changed and a reaction asks for it. Thus, these computed values are at your disposal, apart from the state itself, for your view layer later on. In addition, they don't compute actively every time but rather only compute reactively when a reaction demands it. You can experiment with it in the [MobX Playground](https://jsbin.com/tapiyuricu/1/edit?js,console).

# MobX in React

The basics in MobX should be clear by now. The state in MobX is mutable and can be mutated directly, by actions too or only by actions, and not directly, when enforcing actions with a configuration. When scaling your state management in MobX, you would keep it in multiple yet manageable stores to keep it maintainable. These stores can expose actions and computed values, but most important they make their properties observable. All of these facts already give you an opinionated way of how to store state (e.g. with stores) and how to update the state (e.g. explicit actions with enforced actions). However, you can decide to use a different opinionated approach.

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

Now you have one initial rendering of the view layer, because the `autorun` is running once initially, and successive renderings when the MobX state updates. You can play around with it in the [MobX Playground](https://jsbin.com/delohuwidi/1/edit?js,output).

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

Again you can play around with it in the [MobX Playground](https://jsbin.com/rusewogaza/1/edit?html,js,output). If you would want to use the `TodoList` component as functional component, you can use the `observer` as a function and not as a JavaScript decorator.

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

You can find the example in the following [MobX Playground](https://jsbin.com/wefoyocutu/1/edit?html,js,output) to play around with it.

## Local State

As mentioned before, MobX is not opinionated about how to store your state and how to update it. It goes so far, that you can even exchange your local state in React, `this.state` and `this.setState()`, with MobX. There you wouldn't use a store which separates the state from the view, but use class properties of the component instead. The case can be demonstrated by introducing a component that adds a todo item to the list of todo items from the previous example.

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

  onChange = (event) => {
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

  onSubmit = () => {
    this.props.todoStore.addTodo({
      id: this.id,
      name: this.input,
      completed: false,
    });

    this.id++;
    this.input = '';
  }
# leanpub-end-insert

  onChange = (event) => {
    ...
  }

  render() {
    ...
  }
}
~~~~~~~~

MobX is able to take over the local state of React. You wouldn't need to use `this.state` and `this.setState()` anymore. The previous example can be found in the [MobX Playground](https://jsbin.com/velihicomu/2/edit?html,js,output). Again you experience that MobX isn't opinionated about the way the state is managed. You can have the state management encapsulated in a store class or couple it next to a component as local state. It can make React local state obsolete. But should it? That is up to you.

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

You can open up the application in the [MobX Playground](https://jsbin.com/qecawoweja/2/edit?js,console,output). When you add a todo item, you will get the `console.log()` outputs for the `TodoList` component and only the newly created `TodoItem`. When you complete a todo item, you will only get the `console.log()` of the completing `TodoItem` component. The reactive component only updates when their observable state changes. Everything else doesn't update, because the `observer` decorator implements under the hood the `shouldComponentUpdate()` lifecycle method of React to prevent the component from updating when nothing has changed. You can read more about optimizing MobX performance in React in the [official documentation](https://mobx.js.org/best/react-performance.html).

## Inject Stores

So far, the application passes down the store from the React entry point via props to its child components. They are already passed down more than one layer. However, the store(s) could be used directly in the components by using them directly (when accessible in the file). They are only observable state. Since MobX is not opinionated about where to put state, the observable state, in this case stores, could live anywhere. But as mentioned, the book tries to give an opinionated approach as best practice.

The [mobx-react](https://github.com/mobxjs/mobx-react) library provides you with two helpers to pass the observable state implicitly down to the components (via React`s context) rather than passing them through every component layer explicitly.

The first helper is the `Provider` component that passes down all the necessary observable states down.

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

The second helper from the library is the `inject` decorator. You can use it for any component down your component tree that is wrapped somewhere above by the `Provider` component. It retrieves the provided observable state from React's context as props.

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

Every component can access the observable state, that is passed to the `Provider` component, with the `inject` decorator. This way you keep a clear separation of state and view layer. You can access the project in the [MobX Playground](https://jsbin.com/xubewezeji/3/edit?js,output) again.

## Advanced MobX

MobX is not opinionated. Thus it gives you a handful of tools to accomplish your on way of mastering state management. It would be sufficient to use the basics of MobX to introduce state management in your application. But there are more tools hidden in MobX that this chapter is going to point out. It addition, this chapter should give you a couple more pillars to understand and use MobX successfully in your own way.

### Other Reactions

You have encountered two reactions by now: autorun and observer. The observer produces a reaction because it uses autorun under the hood. It is only used in the mobx-react package. Thus, both functions are used to create reactions based on observable state changes. While the autorun function can be used to re-render naively the UI, it can also used for broader domains. The observer decorator is solely used to make a view layer reactive.

However, MobX comes with more reactions. The book will not go too much into detail here, but it does no harm to be aware of other options too. The [MobX when](https://mobx.js.org/refguide/when.html) is another function that produces a reaction. It is based on predicates and effects. A given predicate runs as long as it returns true. When it returns true, the effect is called. After that the autorunner is disposed. The `when` function returns a disposer to cancel the autorunner prematurely, so before an effect can be called.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const { observable, autorun, computed, when } = mobx;

class TodoStore {
  @observable todos = [];

  constructor() {
    when(
      // once (predicate)...
      () => this.hasCompleteTodos,
      // ... then (effect)
      () => this.celebrateAccomplishment()
    );
  }

  @computed get completeTodos() {
    return this.todos.filter(todo => todo.completed);
  }

  @computed get hasCompleteTodos() {
    return this.completeTodos.length > 0;
  }

  celebrateAccomplishment() {
    console.log('First todo completed, celebrate it!');
  }
}

const todoStore = new TodoStore();

autorun(() => console.log(todoStore.todos.length));

todoStore.todos.push({ id: '0', name: 'finish the book', completed: false });
todoStore.todos.push({ id: '1', name: 'learn redux', completed: true });
todoStore.todos.push({ id: '2', name: 'learn mobx basics', completed: true });
todoStore.todos.push({ id: '3', name: 'learn mobx', completed: false });
~~~~~~~~

So how many times does the reaction run? You can take your guess first and afterward open the [MobX Playground](https://jsbin.com/kuzadiwagi/1/edit?js,console) to experience the reaction yourself. Basically the `when` triggers its effect when the predicate returns true. But it only triggers once. As you can see, two todo items that are completed are added. However, the `celebrateAccomplishment()` method only runs once. This way MobX allows you to use its reactions to trigger side-effects. You could trigger anything ranging from an animation to an API call.

Another function in MobX, [the reaction function](https://mobx.js.org/refguide/reaction.html) itself, can be used to produce MobX reactions too. It is a fine-grained version of autorun. Whereas autorun will always run when the observable state has changed, the reaction function only runs when a particular given observable state has changed.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const { observable, autorun, computed, reaction } = mobx;

class TodoStore {
  @observable todos = [];

  constructor() {

    reaction(
      () => this.completeTodos.length,
      sizeCompleteTodos => console.log(sizeCompleteTodos + " todos completed!")
    );
  }

  @computed get completeTodos() {
    return this.todos.filter(todo => todo.completed);
  }
}

const todoStore = new TodoStore();

autorun(() => console.log(todoStore.todos.length));

todoStore.todos.push({ id: '0', name: 'finish the book', completed: false });
todoStore.todos.push({ id: '1', name: 'learn redux', completed: true });
todoStore.todos.push({ id: '2', name: 'learn mobx basics', completed: true });
todoStore.todos.push({ id: '3', name: 'learn mobx', completed: false });
~~~~~~~~

How many times does the reaction run? First you can have a guess, afterward you can confirm it by trying it in the [MobX Playground](https://jsbin.com/jizidoyoge/1/edit?js,console).

Now you have seen two more functions in MobX that produce reactions: `when` and `reaction`. Whereas the `when` function only runs once an effect when the predicate returns null, the `reaction` runs every time when a particular observable state has changed. You can use both to trigger side-effects, such as an API call.

### Be Opinionated

MobX gives you all the tools that are needed to manage state in modern JavaScript application. However, it doesn't give you an opinionated way of doing it. This way you have all the freedom to manage your state yet it can be difficult to follow best practices or to align a team on one philosophy. That's why it is important to find your own opinionated way of doing things in MobX. You have to align on one opinionated way to manage your state.

The chapters before have shown you that observable state in MobX can be far away managed in stores yet it could be used in the local state of the view layer too. Should MobX be used instead of `this.state` and `this.setState()` in React? Be clear about how close you want to keep your MobX state to your view layer.

Another thing you should have an opinion about is how you update your observable state. Do you mutate the state directly in your view? Going this path would lead to coupling your state closer to your view layer. On the other hand, you could use explicit MobX actions. It would keep your state mutation at one place. You can make them even mandatory by using the `configure()` functionality with enforced actions. That way, every state mutation would have to go through an explicit action. No direct mutations of the state would be allowed anymore. Recommendation: You should make your state mutations as explicit as possible with actions, `configure()` and the `enforceActions` flag set to true.

When using MobX to complement your view layer, you would need to decide on how to pass your state around. You can simply allocate your state next to your components, import it directly from another file using JavaScript ES6 import and export statements, pass it down explicitly (e.g. in React with props) or pass it down implicitly from your root component with `inject()` function and the `Provider` component. You should avoid to mix up these things and follow one opinionated way. Recommendation: You should use the `inject()` function and `Provider` component to make your state implicitly accessible to your view layer.

Last but not least, you would need to align on a state structure. Observable state in MobX can be anything. It can be primitives, it can be objects or arrays but it can also be store instances derived from JavaScript classes. Without mixing up everything, you would need to align on a proper state architecture. The approach to manage your state in stores, as shown in the previous chapters, gives you a maintainable way to manage your state for specific domains. In addition, you are able to keep actions, computed values and even reactions such as autorun, reaction and when in your store. Recommendation: You should use JavaScript classes to manage your state in stores. That way your state management stays maintainable by domain related stores as stakeholders.

As you can see, there are a handful of decisions to make on how to use MobX. It gives you all the freedom to decide your own way of doing things, but after all you have to establish the opinionated way yourself and stay disciplined with it.

## Alternative to Redux?

After all, is MobX a viable alternative to Redux? It depends all on yourself and your requirements. Both solutions are different in their philosophy, their underlying mechanics and in their usage. Whereas Redux gives you one opinionated way of managing your state, MobX gives you only the tools to manage your state but not the way of how to do things. Redux has a large community, a vibrant ecosystem of libraries and great selection of best practices. MobX is a younger library compared to Redux, but gives you a different approach of managing your state and comes with lots of powerful features too.

The defining powers of MobX come from its reactive nature. As you have seen when you connected your view layer to MobX with observers, only the reactive components updated that relied on an observable state change. Everything else stayed untouched. In a large scale application, it can keep your view layer updates to a minimum when using MobX the right way.

In MobX you don't need to normalize your state. You can work with references and keep your state nested. It stays simple to update your state with mutations not worrying about immutability. On the other hand, you have to be cautious on how close you couple your state to your view layer. In the end, when the state is too close to your view layer, it could end up the same way as for the first generation of single page applications were two-way data binding became a mess.

If you want to read more about the differences of Redux and MobX, I recommend you to check out the following article: [Redux or MobX: An attempt to dissolve the Confusion](https://www.robinwieruch.de/redux-mobx-confusion/). After that you might come to a more informed decision on what you want to use for state management in your own application.
