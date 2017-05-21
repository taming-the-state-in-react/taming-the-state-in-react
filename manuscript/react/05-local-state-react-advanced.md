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

- TODO http://stackoverflow.com/questions/43428456/do-i-need-to-use-setstatefunction-overload-in-this-case/43440790#43440790

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

- TODO read for local state with HOC: https://medium.com/airbnb-engineering/rearchitecting-airbnbs-frontend-5e213efc24d2

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