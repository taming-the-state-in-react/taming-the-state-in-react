# Local State Management in React

This chapter will guide you through state management in React without taking any external state management library into account. You will revisit state management in React with `this.setState()` and `this.state`. It enables you already to build large applications without complating and overengineering it.

In the beginning the book tries to come up with defintions to build up on a common vocabulary for state management. It shoulw help you to follow the book more easily when reading it without leaving space for confusion.

By revisiting the topic of local statement in React, you will get to know how to use the local state in React. The chapter makes also sure that you understand the controlled components to leverage the principle of a single source of truth. In addition, you will be taught about the overarching philosophy of the unidirectional data flow in the React ecosystem.

After revisiting the local state, you will get to know best practices and patterns to scale your state management. Even though you are not dependent on an external state management solution, you can make use of those practices to scale your local state management by using `this.setState()` and `this.state` only.

At the end of the chapter, you will get to know the limits of local state management in React. The chapter revisits topics that you might already know from other React learning ressources. It gives you an introduction to the local state by revisiting the usage, the best practices and patterns, and concludes in the scaling problem of

# Defitnions in State Management

In the beginning, I want to give you introductory defintions to state in modern applications. With these definitions at your hands, I want to avoid that you get confused while reading the book.

## State

State is a broad word in modern applications. When speaking about application state, it could be anything that needs to live and be modified in the browser. It could be data that was retrieved from a backend application or a view state in the application, for instance, when toggling a popup to show additional information.

Sometimes I will refer to the former one as **entity state** and to the latter one as **view state**.

When speaking about managing the state, meaning initializing, modifying and deleting state, it will be coined under the umbrella term of state management. Yet state management is a much broader topic. While the mentioned actions are low-level operations, almost implementation details, the architecture, best practices and patterns around state management stay abstract. **State management** invoves all these different topics.

## The Size of State

State can be an atomic object or one global object. When speaking about the view state that only determines if a popup is open or closed, it is an **atomic state object**. When the whole application state can be derived from on object, which includes all the atomic state objects, it is called a **global state object**.

For instance, in games most often the whole application state, the global state object, is called a game object. You can derive the whole state for your game from it. It can be the position of your character in a role play game but also your inevntory of items of your character. Imagine that you only need to lead the application itself and use one global state object to derive everything you need for your application. In a later chapter, you will get to know how this can help for server-side rendering. (TODO check if you really do it!)

The state can be differentiated into **local state** and **sophisticated state**. The management of this state is called **local state management** and **sophisticated state management**.

## Local State

The naming local state is widely accepted in the web development community. Sometimes it can be referenced as internal component state.

Local state is bound to components. It is not stored somewhere else like in sophisticated state management. You will learn about the sophisticated state later. In React the local state is embraced by using `this.state` and `this.setState()`. But it can have a different implementation and usage in other SPA solutions.

## Sophisticated State

I cannot say that it is widely agreed on to call it sophisticated state in the web development dommunity. However, at some point you need a term to distinguish it from local state. That's why I often refer to it as sophisticated state. In other resources you might find it reffered as external state, because it lives outside of the local component.

- as mentioned, it the topic of sophisticated state management with Redux and MobX will come up in later chapters of the book
- sophistacted state management needs to be applied at some point to scale your application
- otherwise you will loose control over your state, because it grows through each component

# Local State in React

How does local state look like in a React component?

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Counter extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0
    };
  }

  render() {
    return (
      <div>
        <p>{this.state.counter}</p>
      </div>
    );
  }
}
~~~~~~~~

The example shows a `Counter` component that has a `counter` property in the state object. It is initiliazed with the value 0 when the component gets instantiated. The `counter` property from the local state object is used in the render method of the component to show the current value.

There is no state manipulation in place yet. You are never allowed to alter the state directly: `this.state.counter = newValue`. You have to use the React component API to change to state explicitly by using: `this.setState()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Counter extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0
    };
  }

  onIncrement = () => {
    this.setState({
      counter: this.state.counter + 1
    });
  }

  onDecrement = () => {
    this.setState({
      counter: this.state.counter - 1
    });
  }

  render() {
    return (
      <div>
        <p>{this.state.counter}</p>
        <button type="button" onClick={this.onIncrement}>
          Increment
        </button>
        <button type="button" onClick={this.onDecrement}>
          Decrement
        </button>
      </div>
    );
  }
}
~~~~~~~~

Now the button `onClick` handler should invoke the class methods to alter the state by either incrementing or decrementing the counter value.

The update functionality with `this.setState()` is performing a shallow merge of objects. What does it mean? Imagine you would have the following state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.state = {
  authors: [...],
  articles: [...]
};
~~~~~~~~

When updating the state only partly, for instance the authors by doing `this.setState({ authors: [{ name: 'Robin', id: '1' }] })`, the articles are left intact. It only updates the `authors` without touching the `articles`.

## Stateful and Stateless Components

Local state can only be used in React ES6 class components. The component becomes a **stateful component** when state is used. Otherwise it can be called **stateless component**.

On the other hand, **functional stateless components** have no state, because, like the name implies, they are only functions and thus are stateless. In a statless component state can only be passed as props from a parent component. In addition, callbacks could be passed to alter the state in the parent component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

function CounterPresenter(props) {
  return (
    <div>
      <p>{props.counter}</p>
      <button type="button" onClick={props.onIncrement}>
        Increment
      </button>
      <button type="button" onClick={props.onDecrement}>
        Decrement
      </button>
    </div>
  );
}
~~~~~~~~

Now a value from state above would be presented in a functional stateless component. However, the functional stateless component is not aware if the passed props are state, props or some other derived value from above. The origin of the props doesn't need to be in the parent component at all, it could be somewhere higher up the component tree.

After all, the callback functions in the stateless component would make it possible to alter the state somewhere above. Once the state was manipulated somewhere above, the new state flows down into the child component as props again. The new value would be displayed correctly.

That's one simple example how local state from one component can traverse down the component tree. To make the example with the functional stateless component complete, let's quickly show how a potential parent component would look like. It is obviously a React ES6 class component, because it has to manage the local state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class CounterContainer extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0
    };
  }

  onIncrement = () => {
    this.setState({
      counter: this.state.counter + 1
    });
  }

  onDecrement = () => {
    this.setState({
      counter: this.state.counter - 1
    });
  }

  render() {
    return <CounterPresenter
      counter={this.state.counter}
      onIncrement={this.onIncrement}
      onDecrement={this.onDecrement}
    />
  }
}
~~~~~~~~

It is not by accident that the suffixes in the naming of both `Counter` components is `Container` and `Presenter`. It is called the [container and presentational component pattern](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). If you are not aware of it, you should definetly read about it. It is a widely used patetrn, where the container component deals with "How things work" and the presenter component deals with "How things look".

## Props vs. State

The previous example made clear that there is a difference between state and props. When properties are passed to a child component, whether it is state, props or derived values, the child component isn't aware of the kind. It sees the incoming properties as props. That's perfect, because the component shouldn't care at all about the kind of properties. It should only make use of them as simple props.

The props come from a parent component. In the parent component these props can be state, props or derived values. It depends on the parent component, if it manages the properties itself (state) or if it gets the properties from a parent componnent too (props).

After all, you can't modify props. Props are only properties passed form a parent component. On the other hand, the state lives in the component. You can access it by using `this.state` and modify it by using `this.setState()`.

When one of these objects changes, whether it is the props that come from the parent component or the state in the component, the update lifecycle methods of the component will run. One of these lifecycle methods is `render()` that updates your component instance based on the props and state. The correct values will be used and displayed.

When you start to use React, it might be difficult to identifiy props and state. Personally I like the {{% a_blank "React official documentation rules" "https://facebook.github.io/react/docs/thinking-in-react.html" %}} to identifiy state.

* Are the properties passed from the parent component? If yes, the likelihood is high that it isn't state. Although it is possible to save props as state, there are little use cases. It should be avoided to save props as state. Use them as props as they are.

* Are the properties unchanged over time? If yes, they don't need to be stateful, because they don't get modified.

* Are the properties deriveable from local state or props? If yes, you don't need it as state, because you can derive it. If you would allocate extra state, the state has to be managed and can get out of sync when you miss to derive it at some point.

## Form State

A common use case in modern applications is to use HTML forms. For instance, you might need to retrieve user information like a name or credit card number or submit a search query to an external API. Forms are used everywhere in modern applications.

There are two ways to use forms in React. You can use the ref attribute or local state. It is recommended to use the latter approach, because the ref attribute is reserved for only a few use cases. If you want to read about these use cases, I encourage you to read the following article: [When to use Ref on a DOM node in React](https://www.robinwieruch.de/react-ref-attribute-dom-node/).

I will quickly demonstrate how form state can be achieved by using the ref attribute. Afterward, I will refactor the example to use local state which is the best practice.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Search extends React.Component {
  constructor(props) {
    super(props);
  }

  onSubmit = () => {
    const { value } = this.input;

    // do something with the search value
    // e.g. propagate it up to the parent component
    // not relevant to show the use case of the ref attribute
    this.props.onSearch(value);

    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.onSubmit}>
        <input
          ref={node => this.input = node}
          type="text"
        />
        <button type="submit">
          Search
        </button>
      </form>
    );
  }
}
~~~~~~~~

The value from the DOM node gets retrieved by using the reference to the DOM node. The reference is created by using the ref attribute.

Now let's see how to make use of local state to embrace best practices.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Search extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      query: ''
    };
  }

  onChange = (event) => {
    const { value } = event.target;

    this.setState({
      query: value
    });
  }

  onSubmit = () => {
    const { value } = this.state;

    // do something with the search value
    // e.g. propagate it up to the parent component
    this.props.onSearch(value);

    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.onSubmit}>
        <input
          onChange={this.onChange}
          type="text"
        />
        <button type="submit">
          Search
        </button>
      </form>
    );
  }
}
~~~~~~~~

You don't need to make use of the ref attribute anymore. You can solve the problem by using local state only. The exampel demonstrates it with only one input field yet it can be used with multple input fields too.

## Controlled Components

The previous example of using form state with local state has one flaw. It doesn't make use of **controlled components**. Naturally a HTML input fields holds its own state. When you enter a value into the input field, the DOM node knows about the value.

However, the value lives in your local state too. You have it in both, the native DOM node state and local state. But you want to make use of a single source of truth. It is a best practice to overwrite the native DOM node state by using the value from the local state.

Let's consider again the previous example. The input field had no value attribute assigned. By using the native value attribute and passing the local state as input, you converted an uncontrolled component to a controlled component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Search extends React.Component {

  ...

  render() {
    return (
      <form onSubmit={this.onSubmit}>
        <input
# leanpub-start-insert
          value={this.state.query}
# leanpub-end-insert
          onChange={this.onChange}
          type="text"
        />
        <button type="submit">
          Search
        </button>
      </form>
    );
  }
}
~~~~~~~~

Now the value comes from the local state as single source of truth. It cannot get out of sync with the native DOM node state.

## Unidirectional Data Flow

In the previous example, you experienced a typically unidirectional data flow. The flux pattern, the underlying architecture for several sophisticated state management solutions, coined the term **unidirectional data flow**. You will get to know more about the flux pattern in a later chapter (TODO later chapter good english? do I really mention the flux pattern again? TODO). But the unidirectional data flow is embraced by local state in React too.

State in React flows only in one direction. State gets updated by using `this.setState()` and is displayed in the render method by accessing `this.state`. Then again it can be updated via `this.setState()`.

The prvious example, where we embraced controlled components, shows the loop of the unidirectional data flow. The input field triggers the `onChange` handler when the input changes. The handler alters the local state. The changed local state triggers an update lifecylce of the component. The update lifecylce runs the render method again. The render method makes use of the updated state. The state flows back to the input field to make it a controlled component. The loop is closed. A new loop can be triggered by typing something in the input field.

The unidirectional data flow makes state management predictable and maintainable. The best practice spread to other libraries and single page application frameworks too. In the previous generation of SPAs most often other mechanics were used.

For instance, in Angular 1.x you had to use two-way data binding in a Model-View-Controller (MVC) architecture. That means, once you changed the value in the view, let's say in an input field by typing something, the value got changed in the controller too. But it worked vice versa too. Once you changed the value in the controller programmatically, the view, to be more specific the inptu field, changed. In a growing application the state became less predictable and maintainable by using two-way data binding.

# Local State in React: Advanced

Now you know all the basics about local state management in React. However, you will notice that there are more patterns and best practices out there. The following chapter gives you insights into these topics.

## Lifting State

After a while, you will notice that you pass quite a lot of state as props down to child components. Yet, the other way around, you will notice that more components need to share the same state. These are two common problems in local state management.

Lifting the state is an operation to keep your state architecture with local state clean. It prevents to share too much or too less state in your component tree. Basically it is a refactoring that you have to do once in a while to keep your components maintainable and focused on only the functionality they were built for.

First, let's dive into an example to demonstrate the uplifting of state. I call it the "Search a List"-example. Second you will experience an example to lift state down. It will be called the "Archive a List"-example.

The "Search a List"-example has two components. A `Search` component and a `List` component. Both components should be used in an overarching `SearchableList` component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Search extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      query: ''
    };
  }

  onChange = (event) => {
    const { value } = event.target;

    this.setState({
      query: value
    });
  }

  render() {
    return (
      <div>
        {this.props.children} <input
          type="text"
          value={query}
          onChange={this.onChange}
        />
      </div>
    );
  }
}

function List({ list }) {
  return (
    <ul>
      {list.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}

function SearchableList({ list }) {
  return (
    <div>
      <Search>Search List:</Search>
      <List list={list} />
    </div>
  );
}
~~~~~~~~

While the `Search` component is a stateful ES6 class component, the `List` component is only a stateless functional component. The parent component that combines the `List` and `Search` components into a `SearchableList` component is a staeless functional component too.

However, the example doesn't work. The `Search` component knows about the `query` that could be used to filter the list, but the `List` component doesn't know about it. You have to lift the state of the `Search` component up to the `SearchableList` to make the `query` state accessible for the `List` component.

In order to lift the state up, the `SearchableList` becomes a stateful component. You have to refactor it to an React ES6 class component. On the other hand, you can refactor the `Search` component to a functional stateless component, because it doesn't need to be stateful anymore. The parent component takes care about it.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

function Search({ query, onChange, children }) {
  return (
    <div>
      {children} <input
        type="text"
        value={query}
        onChange={onChange}
      />
    </div>
  );
}

function List({ list }) {
  return (
    <ul>
      {list.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}

class SearchableList extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      query: ''
    };
  }

  onChange = (event) => {
    const { value } = event.target;

    this.setState({
      query: value
    });
  }

  render() {
    const { list } = this.props;
    const { query } = this.state;
    return (
      <div>
        <Search
          query={query}
          onChange={this.onChange}
        >
          Search List:
        </Search>
        <List list={list.filter(byQuery(query))} />
      </div>
    );
  }
}

function byQuery(query) {
  return function(item) {
    return !query ||
      item.name.toLowerCase().includes(query.toLowerCase());
  }
}
~~~~~~~~

After you have lifted the state up, the parent component takes care about the local state management. Both child components don't need to take care about it anymore.

Let's get to the second example: the "Archive a List"-example. It builds up on the previous example, but this time the `List` component has a functionality to make single items archiveable.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

function Search({ query, onChange, children }) {
  ...
}

# leanpub-start-insert
function List({ list, onArchive }) {
# leanpub-end-insert
  return (
    <ul>
      {list.map(item =>
        <li key={item.id}>
          <span>
            {item.name}
          </span>
# leanpub-start-insert
          <span>
            <button
              type="button"
              onClick={() => onArchive(item.id)}
            >
              Archive
            </button>
# leanpub-end-insert
          </span>
        </li>
      )}
    </ul>
  );
}

class SearchableList extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      query: '',
# leanpub-start-insert
      archivedItems: []
# leanpub-end-insert
    };
  }

  onChange = (event) => {
    const { value } = event.target;

    this.setState({
      query: value
    });
  }

# leanpub-start-insert
  onArchive = (id) => {
    const { archivedItems } = this.state;

    this.setState({
      archivedItems: [...archivedItems, id]
    });
  }
# leanpub-end-insert

  render() {
    const { list } = this.props;
    const { query, archivedItems } = this.state;

    const filteredList = list
      .filter(byQuery(query))
      .filter(byArchived(archivedItems));

    return (
      <div>
        <Search
          query={query}
          onChange={this.onChange}
        >
          Search List:
        </Search>
        <List
# leanpub-start-insert
          list={filteredList}
          onArchive={this.onArchive}
# leanpub-end-insert
        />
      </div>
    );
  }
}

function byQuery(query) {
  return function(item) {
    return !query ||
      item.name.toLowerCase().includes(query.toLowerCase());
  }
}

# leanpub-start-insert
function byArchived(archivedItems) {
  return function(item) {
    return !archivedItems.includes(item.id);
  }
}
# leanpub-end-insert
~~~~~~~~

The previous example was extended to faciliate the archiving of items in a list. Now, the `List` component receives all the neccessary properties, an `onArchive` callback and the list, filtered by `query` and `archivedItems`.

You might see already the flaw. The `SearchableList` takes care about the archiving functionality, even though it doesn't need it by itself. It only passes all the state to the `List` component. Now it makes more sense to lift the state down to the `List` component. Even though the `List` component becomes a stateful component afterward, it is a neccassary step to keep the local state clean.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

function Search({ query, onChange, children }) {
  ...
}

# leanpub-start-insert
class List extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      archivedItems: []
    };
  }

  onArchive = (id) => {
    const { archivedItems } = this.state;

    this.setState({
      archivedItems: [...archivedItems, id]
    });
  }

  render() {
    const { list } = this.props;
    const { archivedItems } = this.state;

    const filteredList = list
      .filter(byArchived(archivedItems));

    return (
      <ul>
        {filteredList.map(item =>
          <li key={item.id}>
            <span>
              {item.name}
            </span>
            <span>
              <button
                type="button"
                onClick={() => onArchive(item.id)}
              >
                Archive
              </button>
            </span>
          </li>
        )}
      </ul>
    );
  }
}

class SearchableList extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      query: ''
    };
  }

  onChange = (event) => {
    const { value } = event.target;

    this.setState({
      query: value
    });
  }

  render() {
    const { list } = this.props;
    const { query } = this.state;

    const filteredList = list
      .filter(byQuery(query));

    return (
      <div>
        <Search
          query={query}
          onChange={this.onChange}
        >
          Search List:
        </Search>
        <List list={filteredList} />
      </div>
    );
  }
}
# leanpub-end-insert

function byQuery(query) {
  return function(item) {
    return !query ||
      item.name.toLowerCase().includes(query.toLowerCase());
  };
}

function byArchived(archivedItems) {
  return function(item) {
    return !archivedItems.includes(item.id);
  };
}
~~~~~~~~

Lifting the state down wouldn't make sense if another component would be interested in the `archivedItems`.

Now, you have seen both variations of lifting state: lifting state up and lifting state down.

In the first example, the "Search a List"-example, the state had to be lifted up to share the `query` property in two child components. The `Search` component had to manipulate the state by using a callback, but also had to use the `query` to be a controlled component. On the other hand, the 'SearchableList' component had to filter the list by using the `query` property. Another solution would have been to pass down the `query` property to the `List` component and let the component deal itself with the filtering.

in the second example, the "Archive a List"-example, the state had to be lifted down. The parent component shouldn't be concerned about state that isn't used by the parent component itself and isn't shared across multiple child components. Because only one child component cared about the archived items, it was a neccassry refactoring to lift the state down.

In conclusion, lifting state allows you to keep your local state management maintainable. **Lifting state should be used to give components access to all the state they need, but not to more state than they need.**

Sometimes you have to refactor components from a functional stateless component to a React ES6 class component or vice versa. It's not always possible, because a component that could become possibly a stateless functional component could still have other stateful properties.

## Functional State

In all recent chapters, I made a mistake when using `this.setState()`. Let me first explain what's wrong about its usage and second tell you why I didn't correct it immediately.

It is important to know that `this.setState()` is executed asynchronously. React batches all the state updates. It executes them after each other for performance optimization.

In its first version, the `this.setState()` method takes an object to update the state. The merging of the object is shallow. For instance, when updating `authors` in a state object of `authors` and `articles`, the `articles` stay intact.

The previos example have used this approach already.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState({
  ...
});
~~~~~~~~

In its second version, the `this.setState()` method takes a function. The function has the previous state and props at its disposal in the function signature.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState((prevState, props) => ({
  ...
}));
~~~~~~~~

What's the flaw in using `this.setState()` with an object? In several examples in the last chapters, the state was updated based on the previous state or props. But since `this.setState()` executes asynchronously, the state or props usedm when the method is finally executed, could be stale properties. It could lead to bugs in your local state management, because you would update the state with stale properties. When using the functional approach to update the state, the correct state and props are used when the function executes asynchronously.

Why didn't I correct the mistake immediately?

First, it makes sense to know about your options. Would I have corrected it immediatly, you wouldn't have interlaized the `this.setState()` approach with an object. You would always default to the functional approach. However, the fact that `this.setState()` executes asynchronously is important. You should not miss this learning and the delayed explaination of this fact made you probably more aware of it.

After all you might wonder, when to use the object and when to use the function in `this.setState()`? The rules of thumb:

* Always use `this.setState()` with a function when you depend on previous state or props.
* Only use `this.setState()` with an object when you don't depend on previous properties.
* In case you are unsure, default to use `this.setState()` with a function.

What are the benefits of using the functional approach of updating the state? Let's revisit one of the recent examples.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class CounterContainer extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0
    };
  }

  onIncrement = () => {
    this.setState({
      counter: this.state.counter + 1
    });
  }

  onDecrement = () => {
    this.setState({
      counter: this.state.counter - 1
    });
  }

  render() {
    return <CounterPresenter
      counter={this.state.counter}
      onIncrement={this.onIncrement}
      onDecrement={this.onDecrement}
    />
  }
}
~~~~~~~~

To make the mistake clear again, executing the `onIncrement` callback multiple times in a row, could already lead to a bug. Because the increment (the decrement too) depends on the previous state, it would use stale state when the asynchronous update wasn't perfomed.

{title="Code Playground",lang="javascript"}
~~~~~~~~
onIncrement() // previous state: { counter: 0 }
onIncrement() // previous state: { counter: 0 }
onIncrement() // previous state: { counter: 0 }
// updated state: { counter: 1 }
// instead of: { counter: 3 }
~~~~~~~~

This becomes more error prone when multiple `this.setState()` methods depend on the previous state and are implcitly connected to other sub states.

Refactoring the example to use the functional approach would fix the bug.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class CounterContainer extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0
    };
  }

  onIncrement = () => {
    this.setState(prevState => ({
      counter: prevState.counter + 1
    }));
  }

  onDecrement = () => {
    this.setState(prevState => ({
      counter: prevState.counter - 1
    }));
  }

  render() {
    return <CounterPresenter
      counter={this.state.counter}
      onIncrement={this.onIncrement}
      onDecrement={this.onDecrement}
    />
  }
}
~~~~~~~~

The functional approach opens up two more benefits. First, the function that updates the state is pure. There are no side-effects. The function always will return the same output when given the same input. It makes it predicable and uses the benefits of functional programming. Second, since the function is pure, it can be tested easily in an unit test and independently from the component. It gives you the opportunity to test your local state updates. You only have to extract the function from the component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

# leanpub-start-insert
const incrementUpdate = prevState => ({
  counter: prevState.counter + 1
});

const decrementUpdate = prevState => ({
  counter: prevState.counter - 1
});
# leanpub-end-insert

class CounterContainer extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0
    };
  }

# leanpub-start-insert
  onIncrement = () => {
    this.setState(incrementUpdate);
  }

  onDecrement = () => {
    this.setState(decrementUpdate);
  }
# leanpub-end-insert

  render() {
    return <CounterPresenter
      counter={this.state.counter}
      onIncrement={this.onIncrement}
      onDecrement={this.onDecrement}
    />
  }
}
~~~~~~~~

Now you could export the pure functions to test them. After all, that's what makes the functional approach so powerful. It could lead to an shift of how we update local state. The default approach suggested by the official documentation is using `this.setState()` with an object. But in the near future the default could shift towards using the functional approach.

## Higher Order Components for Local State Management

Higher order components (HOCs) can be used for a handful of use cases. One use case would be to [enable an elegant way of conditional rendering](https://www.robinwieruch.de/gentle-introduction-higher-order-components/). But this book is about state management, so why not use it do manage the local state of a component?

Let's revisit an adjusted example of the `Archive a List`-example.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class ArchiveableList extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      archivedItems: []
    };
  }

  onArchive = (id) => {
    const { archivedItems } = this.state;

    this.setState({
      archivedItems: [...archivedItems, id]
    });
  }

  render() {
    const { list } = this.props;
    const { archivedItems } = this.state;

    const filteredList = list
      .filter(byArchived(archivedItems));

    return (
      <ul>
        {filteredList.map(item =>
          <li key={item.id}>
            <span>
              {item.name}
            </span>
            <span>
              <button
                type="button"
                onClick={() => onArchive(item.id)}
              >
                Archive
              </button>
            </span>
          </li>
        )}
      </ul>
    );
  }
}

function byArchived(archivedItems) {
  return function(item) {
    return !archivedItems.includes(item.id);
  };
}
~~~~~~~~

The `ArchiveableList` has two purposes. On the one hand, it is a pure presenter that shows the items in a list. On the other hand, it is stateful container that keeps track of the archived items. Obviously you could split it up into representation and logic thus into presentational and container component. Another approach could be to transfer the logic, mainly the local state management, into a higher order component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

function byArchived(archivedItems) {
  return function(item) {
    return !archivedItems.includes(item.id);
  };
}

function withArchive(Component) {
  class WithArchive extends React.Component {
    constructor(props) {
      super(props);

      this.state = {
        archivedItems: []
      };
    }

    onArchive = (id) => {
      const { archivedItems } = this.state;

      this.setState({
        archivedItems: [...archivedItems, id]
      });
    }

    render() {
      const { list } = this.props;
      const { archivedItems } = this.state;

      const filteredList = list
        .filter(byArchived(archivedItems));

      return <Component
        list={filteredList}
        onArchive={this.onArchive}
      />
    }
  }

  return WithArchive;
}

function List({ list, onArchive }) {
  return (
    <ul>
      {list.map(item =>
        <li key={item.id}>
          <span>
            {item.name}
          </span>
          <span>
            <button
              type="button"
              onClick={() => onArchive(item.id)}
            >
              Archive
            </button>
          </span>
        </li>
      )}
    </ul>
  );
}
~~~~~~~~

Now you can compose a list facilitating component with the functionality to archive items in a list.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

function byArchived(archivedItems) {
  ...
}

function withArchive(Component) {
  ...
}

function List({ list, onArchive }) {
  ...
}

const ListWithArchive = withArchive(List);

function App({ list }) {
  return <ListWithArchive list={list} />
}
~~~~~~~~

# Persistence in State

You might wonder how to persist the local state? The question applies for local state management, but in the following also for sophisticated state management.

Obviously you would need a backend with a database to store the state. Extracting the state from your application is called **dehydrating state**. Now, every time your application bootstraps, you would retrieve the state from the backend that keeps it in a database. Once the state arrives asynchronously in your request, you would **rehydrate state** into your application.

While the dehydration of the state could happen any time your application is running, the rehydration would take place when your components mount. The best place to do it would be the `componentDidMount()` lifecycle method. Take for example the `ArchiveableList` component. It could retrieve all the already archived ids of items in the when mounting and rehydrating it to the local state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class ArchiveableList extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      archivedItems: []
    };
  }

  onArchive = (id) => {
    ...
  }

  componentDidMount() {
    fetch('path/to/archived/items')
      .then(response => response.json())
      .then(archivedItems => this.setState(rehydrateArchivedItems(archivedItems)));
  }

  render() {
    ...
  }
}

function rehydrateArchivedItems(archivedItems) {
  return function(prevState) {
    return {
      archivedItems: [
        ...prevState.archivedItems,
        ...archivedItems
      ]
    };
  };
}
~~~~~~~~

Now, every time the component initializes, the persistent archived items will get rehydrated into the application state.

The dehydration could happen anytime, but to avoid inconstencies, in the example of archived items, the dehydration would take place when an item gets archived. It is an usual request to the backend to save the item as being archived.

The rehydration and dehydration of state are most often unconcious steps in modern applications. It is common sense to retrieve all the necessary data from the backend when your application bootstraps and to update the data when something has changed. But you can keep the rehydration and dehydration of state in mind to keep your application state in sync with your backend data as single source of truth.

Is there a more lightweight solution compared to a backend application? You could use the native browser API. To be more specific, most of the modern browser have a storage functionality to persist data. It is the lightwight version of a database in the browser. Obviously, it is only visible to the user of the browser and cannot be distributed to other users.

Modern browsers have access to the [local storage](https://developer.mozilla.org/en/docs/Web/API/Window/localStorage) and [session storage](https://developer.mozilla.org/en/docs/Web/API/Window/sessionStorage). Both work the same, but there is one difference in their functionalities. While the local storage keeps the data even when the browser is closed, the session storage expires once the browser closes.

Both storages work by using key value pairs.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// Save data to localStorage
localStorage.setItem('key', 'value');

// Get saved data from localStorage
var data = localStorage.getItem('key');

// Remove saved data from localStorage
localStorage.removeItem('key');

// Remove all saved data from localStorage
localStorage.clear();
~~~~~~~~

You can substitute the `localStorage` with the `sessionStorage`. In the end, you can apply them the same way as you did in the previous `ArchiveableList` component that used the backend request to retrieve the data. Only that the `ArchiveableList` component would uss the storage instead of the backend.

# Caching in State

Same as the strategy for keeping a persistent state, the caching applies as well for sophisticated state management. But the following will demonstrate it by using local state management.

Imagine your application has an interface to search for popular stories on a news platform. The news platform has an open API that you can use to retrieve those popular stories. Your own application only has a search field and a list of popular stories once they have been searched.

Next imagine that you make you first request to search about "React". You are not satisfied with the search result, because you wanted to be more specific, and search again for "Redux". Still, no satisfying search result. The search result for "Redux" is still visible in your application. Now you want to head back to search for "React" stories again. Your application makes a third request to retrieve the "React" stories. But you know that you already searched for it before. That's where caching comes into play. The third request could be avoided when the application would have cached the search results.

Such a fluctuant cache solution is not difficult to implement with a local state. Bear in mind that it would work out with a sophisticated state too. When searching for the stories, you already have an unique identifier which you can use as a key in an object to store the search result. The unique identifier is your search term. It would be either "React" or "Redux" when considereing the previous example. The value corresponending to the key would be the search result. In the example it would be the popular stories. After all, your cache object in the local state would look similar to this:

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.state = {
  ...
  searchCache: {
    react: [...],
    redux: [...],
  }
}
~~~~~~~~

Every time your application performs a search request, the key value pair in the cache object in your local state would be filled. Before you make a new request, the cache would be checked if the search term is already available as a key. If the key is available, the request would be surpressed and the cache result would be used instead. If the key is not available, a request would be made. After the request succeeded, the search term would be saved as key and the search result would be saved as value for the key in the local state.

The book doesn't give you an in-depth implementation of the cache solution. If you did read [the Road to learn React](https://www.robinwieruch.de/the-road-to-learn-react/), you will already know how to implement such a cache. In one of its lessons, the book uses a cache in a more elaborated way to cache paginated search results efficiently.

# The Lies of Local State Management

State management is a controversial topic. You will find a ton of discussion and opinions around it. You will find it as reoccuring topic not only in React, but also in other solutions for modern applications. The book is my attempt to give you consistency for these opinions and enable you to learn state management step by step.

The following statement is controversial: *The local state in React is sufficient for most of your application. You will not need a sophisticated state managament solutions like Redux or MobX.*

Personally I agree with the statement. You can build quite large applications with local state only. You should be aware of best practices and patterns to scale it, but it is doable. You can spare a lot of application complextity by using local state only. Once your application scales, you might want to apply sophisticaed state management.

The next statement might be controversial too: *Once you have a sophisitcated state management in place, you shouldn't use local state anymore.*

Personally I disagree with the statement. Not every state should live in a sophisticated state management. There are use cases where local state is applicable in large applications. Especially when considering entity state and view state: The view state can most often live in a local state, because it is not shared widely across the application. But the entity state can live in a sophisticated state, because it is shared across multiple components. It might need to be accessible and modifyable by multiple components across your application.

Last but not least, another controversial statement: *You don't need local state, learn Redux instead when you learn React.*

Personally I strongly disagree with the statement. If you want to develop applications with React, you should certainly be aware of local state in React. You should have build applications with it before you start to learn sophisticated state management solutions like Redux. You need to run into local state management problems before you get the help of sophisticated state management solutions.

These were only three controversial statemanemts. But there are way more opinions around the topic. In the end, you should make your own experiences to get to know what makes sense for you.

# The Flaw of Local State Management

In order to come to a conclusion of local state management there is one open question: What's the problem with local state management? Developers wonder why they need sophisticated state management in order to tame their state. In other scenarios people never wonder about it, because they learned sophistiacted state management from the beginnign without using local state. That might be not the best approahc in the first place, because you have to experience a problem before you use a solution for it. You can't skip the problem and use the solution right away.

So what's the problem in local state management? It doesn't scale in large applications. It doesn't scale implementation wise, but it might doesn't scale in a team of developers too.

Implementaiton wise it doesn't scale because too many components across your application share state. They need to access the state, need to modify it or need to remove it. In a small applications these components are not far away from each other. You can apply best practices like lifting state up and down to keep the state management clean. At some point components are too far away from each other. The state needs to be lifted up the component tree all the way up. Still, child components could be multiple levels below the stateful component. The state would creep through all components in between even though these component don't need access to it. It becomes a state soup.

Local state can become unmaintanable. It is already difficult for one person to keep the places in mind where local state is used in the component tree. When a team of developers implements one application, it becomes even more difficult to keep track of it. Usually it is not neccessary to keep track about the local state. In a perfect world, everyone would lift state up and down to keep it clean. In the real world, code doesn't get refactored as often as it should be. The state creeps through all component even though they don't need it.

One could argue that the difficult maintainablity applies for sophisticaed state as well. That's true, there are pitfalls again that people need to avoid to keep the state management maintainable. But at least the state management is gathered at one place to maintain it. It doesn't get too mixed up with the view layer. There are only bridges that connect the view with the state.