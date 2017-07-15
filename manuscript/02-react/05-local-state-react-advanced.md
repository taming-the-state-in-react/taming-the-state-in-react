## Scaling Local State in React

You should know about all the basics in React's local state management by now. However, you will notice that there are more patterns and best practices out there to apply local state in a scaling application. The following chapter gives you insights into these topics.

### Lifting State

In a scaling application, you will notice that you pass a lot of state down to child components as props. These props are often passed down multiple component levels. That's how state is shared vertically. Yet, the other way around, you will notice that more components need to share the same state. That's how state is shared horizontally. These two scaling issues, sharing state vertically and horizontally, are common in local state management. Therefore you can lift the state up and down to keeo your local state architecture maintainable. Lifting the state prevents to share too much or too less state in your component tree. Basically, it is a refactoring that you have to do once in a while to keep your components maintainable and focused on only consuming the state that they need to consume.

In order to experience up and down lifting of local state, the following chapter will demonstrate it with two examples. The first example that demonstrates the uplifting of state is called: "Search a List"-example. The second example that demonstrates the downlifting of state is called "Archive in a List"-example.

The "Search a List"-example has three components. Two sibling components, a `Search` component and a `List` component, that are used in an  overarching `SearchableList` component.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Search extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      query: ''
    };

    this.onChange = this.onChange.bind(this);
  }

  onChange(event) {
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
          value={this.state.query}
          onChange={this.onChange}
        />
      </div>
    );
  }
}
~~~~~~~~

{title="Code Playground",lang="javascript"}
~~~~~~~~
function List({ list }) {
  return (
    <ul>
      {list.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}
~~~~~~~~

{title="Code Playground",lang="javascript"}
~~~~~~~~
function SearchableList({ list }) {
  return (
    <div>
      <Search>Search List:</Search>
      <List list={list} />
    </div>
  );
}
~~~~~~~~

While the `Search` component is a stateful ES6 class component, the `List` component is only a stateless functional component. The parent component that combines the `List` and `Search` components into a `SearchableList` component is a stateless functional component too.

However, the example doesn't work. The `Search` component knows about the `query` that could be used to filter the list, but the `List` component doesn't know about it. You have to lift the state of the `Search` component up to the `SearchableList` to make the `query` state accessible for the `List` component. You want to share the state in both `List` component and `Search` component.

In order to lift the state up, the `SearchableList` becomes a stateful component. You have to refactor it to an React ES6 class component. On the other hand, you can refactor the `Search` component to a functional stateless component, because it doesn't need to be stateful anymore. The stateful parent component takes care about it.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function Search({ query, onChange, children }) {
# leanpub-end-insert
  return (
    <div>
      {children} <input
        type="text"
# leanpub-start-insert
        value={query}
        onChange={onChange}
# leanpub-end-insert
      />
    </div>
  );
}
~~~~~~~~

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class SearchableList extends React.Component {
  constructor(props) {
    super(props);

# leanpub-start-insert
    this.state = {
      query: ''
    };

    this.onChange = this.onChange.bind(this);
  }

  onChange(event) {
    const { value } = event.target;

    this.setState({
      query: value
    });
  }
# leanpub-end-insert

  render() {
    const { list } = this.props;
# leanpub-start-insert
    const { query } = this.state;
# leanpub-end-insert

    return (
      <div>
        <Search
# leanpub-start-insert
          query={query}
          onChange={this.onChange}
# leanpub-end-insert
        >
          Search List:
        </Search>
# leanpub-start-insert
        <List list={(list || []).filter(byQuery(query))} />
# leanpub-end-insert
      </div>
    );
  }
}

# leanpub-start-insert
function byQuery(query) {
  return function(item) {
    return !query ||
      item.name.toLowerCase().includes(query.toLowerCase());
  }
}
# leanpub-end-insert
~~~~~~~~

After you have lifted the state up, the parent component takes care about the local state management. Both child components don't need to take care about it anymore. You have lifted the state up to share the local state across more components. THe list gets filtered by the search query before it reaches the `List` component.

Let's get to the second example: the "Archive in a List"-example. It builds up on the previous example, but this time the `List` component has the extended functionality to archive an item in the list.

{title="Code Playground",lang="javascript"}
~~~~~~~~
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
~~~~~~~~

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class SearchableList extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      query: '',
# leanpub-start-insert
      archivedItems: []
# leanpub-end-insert
    };

    this.onChange = this.onChange.bind(this);
# leanpub-start-insert
    this.onArchive = this.onArchive.bind(this);
# leanpub-end-insert
  }

  ...

# leanpub-start-insert
  onArchive(id) {
    const { archivedItems } = this.state;

    this.setState({
      archivedItems: [...archivedItems, id]
    });
  }
# leanpub-end-insert

  render() {
    const { list } = this.props;
# leanpub-start-insert
    const { query, archivedItems } = this.state;

    const filteredList = list
      .filter(byQuery(query))
      .filter(byArchived(archivedItems));
# leanpub-end-insert

    return (
      <div>
        ...
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

...

# leanpub-start-insert
function byArchived(archivedItems) {
  return function(item) {
    return !archivedItems.includes(item.id);
  }
}
# leanpub-end-insert
~~~~~~~~

The previous example was extended to faciliate the archiving of items in a list. Now, the `List` component receives all the neccessary properties, an `onArchive` callback and the list, filtered by `query` and `archivedItems`.

You might see already the flaw. The `SearchableList` takes care about the archiving functionality. Howeover, it doesnt need the functionality itself. It only passes all the state to the `List` component. In a scaling application it would make sense to lift the state down to the `List` component. Even though the `List` component becomes a stateful component afterward, it is step in the right direction keeping the local state maintainable in the long run.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

# leanpub-start-insert
class List extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      archivedItems: []
    };

    this.onArchive = this.onArchive.bind(this);
  }

  onArchive(id) {
    const { archivedItems } = this.state;

    this.setState({
      archivedItems: [...archivedItems, id]
    });
  }
# leanpub-end-insert

  render() {
    const { list } = this.props;
# leanpub-start-insert
    const { archivedItems } = this.state;

    const filteredList = list
      .filter(byArchived(archivedItems));
# leanpub-end-insert

    return (
      <ul>
# leanpub-start-insert
        {filteredList.map(item =>
# leanpub-end-insert
          <li key={item.id}>
            <span>
              {item.name}
            </span>
            <span>
              <button
                type="button"
# leanpub-start-insert
                onClick={() => this.onArchive(item.id)}
# leanpub-end-insert
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
~~~~~~~~

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class SearchableList extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      query: ''
    };

    this.onChange = this.onChange.bind(this);
  }

  ...

  render() {
    const { list } = this.props;
# leanpub-start-insert
    const { query } = this.state;

    const filteredList = list
      .filter(byQuery(query));
# leanpub-end-insert

    return (
      <div>
        ...
# leanpub-start-insert
        <List list={filteredList} />
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

Now, you have seen both variations of lifting state: lifting state up and lifting state down.

In the first example, the "Search a List"-example, the state had to be lifted up to share the `query` property in two child components. The `Search` component had to manipulate the state by using a callback, but also had to use the `query` to be a controlled component. On the other hand, the 'SearchableList' component had to filter the list by using the `query` property. Another solution would have been to pass down the `query` property to the `List` component and let the component deal with the filtering.

In the second example, the "Archive in a List"-example, the state could be lifted down to keep the state maintainable in the long run. The parent component shouldn't be concerned about state that isn't used by the parent component itself and isn't shared across multiple child components. Because only one child component cared about the archived items, it was a clean code refactoring to lift the state down.

In conclusion, lifting state allows you to keep your local state management maintainable. **Lifting state should be used to give components access to all the state they need, but not to more state than they need.** Sometimes you have to refactor components from a functional stateless component to a React ES6 class component or vice versa. It's not always possible, because a component that could become possibly a stateless functional component could still have other stateful properties.

### The Provider Pattern

- TODO Provider pattern with context and hocs https://medium.com/@nimelrian/thinking-in-react-a-paradox-statement-33c19e2eb9e2

### Functional State

In all recent chapters, there is a mistake in using `this.setState()`. It is important to know that `this.setState()` is executed asynchronously. React batches all the state updates. It executes them after each other for performance optimization. Thus `this.setState()` comes in two versions.

In its first version, the `this.setState()` method takes an object to update the state. As explained in a previous chapter, the merging of the object is shallow. For instance, when updating `authors` in a state object of `authors` and `articles`, the `articles` stay intact. The previous examples have already used this approach.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState({
  ...
});
~~~~~~~~

In its second version, the `this.setState()` method takes a function. The function has the previous state and props in the function signature.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState((prevState, props) => ({
  ...
}));
~~~~~~~~

So, what's the flaw in using `this.setState()` with an object? In several examples in the last chapters, the state was updated based on the previous state or props. However, `this.setState()` executes asynchronously. Thus the state or props that are used to perform the update could be stale at this point in time. It could lead to bugs in your local state management, because you would update the state based on stale properties. When using the functional approach to update the local state, the state and props at the time of execution are used when `this.setState()` performs asynchronously. Let's revisit one of the previous examples:

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class CounterContainer extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0
    };

    this.onIncrement = this.onIncrement.bind(this);
    this.onDecrement = this.onDecrement.bind(this);
  }

  onIncrement() {
    this.setState({
      counter: this.state.counter + 1
    });
  }

  onDecrement() {
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

Executing one of the class methods, `onIncrement` or `onDecrement`, multiple times could lead to a bug. Because both methods depend on the previous state, it could use a stale state when the asynchronous update wasn't executed but the method invoked a second time.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState({ counter: this.state.counter + 1 }); // this.state: { counter: 0 }
this.setState({ counter: this.state.counter + 1 }); // this.state still: { counter: 0 }
this.setState({ counter: this.state.counter + 1 }); // this.state still: { counter: 0 }
// updated state: { counter: 1 }
// instead of: { counter: 3 }
~~~~~~~~

It becomes even more error prone when multiple functions that use `this.setState()` depend on the previous state. You can refactor the example to use the functional state updating approach.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class CounterContainer extends React.Component {
  constructor(props) {
    ...
  }

# leanpub-start-insert
  onIncrement() {
    this.setState(prevState => ({
      counter: prevState.counter + 1
    }));
  }

  onDecrement() {
    this.setState(prevState => ({
      counter: prevState.counter - 1
    }));
  }
# leanpub-end-insert

  render() {
    ...
  }
}
~~~~~~~~

The functional approach opens up two more benefits. First, the function that updates the state is a pure function. There are no side-effects. The function always will return the same output when given the same input. It makes it predicable and uses the benefits of functional programming. Second, since the function is pure, it can be tested easily in an unit test and independently from the component. It gives you the opportunity to test your local state updates. You only have to extract the function from the component.

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
    ...
  }

# leanpub-start-insert
  onIncrement() {
    this.setState(incrementUpdate);
  }

  onDecrement() {
    this.setState(decrementUpdate);
  }
# leanpub-end-insert

  render() {
    ...
  }
}
~~~~~~~~

Now, you could test the pure functions. After all you might wonder, when to use the object and when to use the function in `this.setState()`? The recommended rules of thumb are:

* Always use `this.setState()` with a function when you depend on previous state or props.
* Only use `this.setState()` with an object when you don't depend on previous properties.
* In case you are unsure, default to use `this.setState()` with a function.

### Higher Order Components

Higher order components (HOCs) can be used for a handful of use cases. One of these use case would be to [enable an elegant way of conditional rendering](https://www.robinwieruch.de/gentle-introduction-higher-order-components/). But this book is about state management, so why not use it do manage the local state of a component? Let's revisit an adjusted example of the "Archive in a List"-example.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class ArchiveableList extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      archivedItems: []
    };

    this.onArchive = this.onArchive.bind(this);
  }

  onArchive(id) {
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

The `ArchiveableList` has two purposes. On the one hand, it is a pure presenter that shows the items in a list. On the other hand, it is stateful container that keeps track of the archived items. Therefore you could split it up into representation and logic thus into presentational and container component. However, another approach could be to transfer the logic, mainly the local state management, into a higher order component.

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

      this.onArchive = this.onArchive.bind(this);
    }

    onArchive(id) {
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
~~~~~~~~

{title="Code Playground",lang="javascript"}
~~~~~~~~
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

Now you can compose a 'List' facilitating component with the functionality to archive items in a list.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

function byArchived(archivedItems) { ... }

function withArchive(Component) { ... }

function List({ list, onArchive }) { ... }

const ListWithArchive = withArchive(List);

function App({ list }) {
  return <ListWithArchive list={list} />
}
~~~~~~~~

The `List` component would only display the items. The ability to archive an item in the `List` component would be opt-in with a higher order component called `withArchive`. In addition, the HOC can be reused in other `List` components too for managing the state of archived items. After all, higher order components are great to extract local state management from components and to reuse the local state management in other components.