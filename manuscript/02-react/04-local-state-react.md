## Local State in React

The book uses React as view layer for demonstrating the local state. The following chapter focuses on the local state in React before it dives into sophistaicated state management with Redux and MobX. As mentioned, the concept of local state should be known in other SPA solutions too and thus applicable in those.

So, how does local state look like in a React component?

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

The example shows a `Counter` component that has a `counter` property in the local state object. It is initiliazed with a value of 0 when the component gets instantiated by its constructor. In addition, the `counter` property from the local state object is used in the render method of the component to display its current value.

There is no state manipulation in place yet. Before you start to manipulate your state, you should know that you are never allowed to alter the state directly: `this.state.counter = 1`. Instead, you have to use the React component API to change the state explicitly by using: `this.setState()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Counter extends React.Component {
  constructor(props) {
    ...

# leanpub-start-insert
    this.onIncrement = this.onIncrement.bind(this);
    this.onDecrement = this.onDecrement.bind(this);
# leanpub-end-insert
  }

# leanpub-start-insert
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
# leanpub-end-insert

  render() {
    ...
  }
}
~~~~~~~~

The class methods can be used in the `render()` method to trigger the local state changes.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Counter extends React.Component {
  ...

  render() {
    return (
      <div>
        <p>{this.state.counter}</p>
# leanpub-start-insert
        <button type="button" onClick={this.onIncrement}>
          Increment
        </button>
        <button type="button" onClick={this.onDecrement}>
          Decrement
        </button>
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

Now the button `onClick` handler should invoke the class methods to alter the state by either incrementing or decrementing the counter value.

The update functionality with `this.setState()` is performing a **shallow merge** of objects. What does a shallow merge mean? Imagine you would have the following state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.state = {
  authors: [...],
  articles: [...],
};
~~~~~~~~

When updating the state only partly, for instance the authors by doing the following, the articles are left intact

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState({
  authors: [
    { name: 'Robin', id: '1' }
  ]
});
~~~~~~~~

It only updates the `authors` without touching the `articles`. That's a shallow merge.

### Stateful and Stateless Components

Local state can only be used in React ES6 class components. The component becomes a **stateful component** when state is used. Otherwise it can be called **stateless component** even though it is a React ES6 class component.

On the other hand, **functional stateless components** have no state, because, like the name implies, they are only functions and thus are stateless. In a statless component state can only be passed as props from a parent component. In addition, callback functions could be passed down to alter the state in the parent component.

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

Now only the props from the parent component would be used in a functional stateless component. The `counter` props would be displayed and the two callback functions, `onIncrement` and `onDecrement` would be used for the buttons. However, the functional stateless component is not aware whether the passed properties are state, props or some other derived properties. The origin of the props doesn't need to be in the parent component after all, it could be somewhere higher up the component tree. The parent component would only pass or manipulate the properties along the way. In addition, the component is unaware of what the callback functions are doing. It doesn't need to know that these alter the local state of the parent component.

After all, the callback functions in the stateless component would make it possible to alter the state somewhere above in one of the parent components. Once the state got manipulated, the new state flows down as props into the child component again. The new `counter` prop would be displayed correctly, because the render method of the child component runs again with the changed props.

That's one example how local state from one component can traverse down the component tree. To make the example with the functional stateless component complete, let's quickly show how a potential parent component, that manages the local state, would look like. It is a React ES6 class component in order to be stateful.

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

It is not by accident that the suffixes in the naming of both `Counter` components are `Container` and `Presenter`. It is called the [container and presentational component pattern](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). If you are not aware of it, you should definetly read about it. It is a widely used patetrn, where the container component deals with "How things work" and the presenter component deals with "How things look". In this case, the container component cares about the state while the presenter component only displays the counter value and provides click handler yet without knowing that these click handlers manipulate the state.

Container components are the ideal candidates to manage state while the presenter components only display it and act on callback functions. You will encounter these container components more often in the book, when dealing with the concepts of higher order components, that could manage local state, and connected component.

### Props vs. State

The previous example made clear that there is a difference between state and props. When properties are passed to a child component, whether it is state, props or derived properties, the child component isn't aware of the kind of properties. It sees the incoming properties as props. That's perfect, because the component shouldn't care at all about the kind of properties. It should only make use of them as simple props.

The props come from a parent component. In the parent component these props can be state, props or derived properties. It depends on the parent component, if it manages the properties itself (state), if it gets the properties from a parent componnent too (props) or if it derives new properties from the props from its parent component along the way (derived properties).

After all, you can't modify props. Props are only properties passed form a parent component. On the other hand, the local state lives in the component itself. You can access it by using `this.state`, modify it by using `this.setState()` and pass it down as props to child components.

When one of these objects changes, whether it is the props that come from the parent component or the state in the component, the update lifecycle methods of the component will run. One of these lifecycle methods is the `render()` method that updates your component instance based on the props and state. The correct values will be used and displayed after the update ran.

When you start to use React, it might be difficult to identifiy props and state. Personally I like the {{% a_blank "React official documentation rules" "https://facebook.github.io/react/docs/thinking-in-react.html" %}} to identify state.

* Are the properties passed from the parent component? If yes, the likelihood is high that it isn't state. Although it is possible to save props as state, there are little use cases. It should be avoided to save props as state. Use them as props as they are.

* Are the properties unchanged over time? If yes, they don't need to be stateful, because they don't get modified.

* Are the properties deriveable from local state or props? If yes, you don't need it as state, because you can derive it. If you would allocate extra state, the state has to be managed and can get out of sync when you miss to derive the new properties it at some point.

### Form State

A common use case in modern applications is to use HTML forms. For instance, you might need to retrieve user information like a name or credit card number or submit a search query to an external API. Forms are used everywhere in modern applications.

There are two ways to use forms in React. You can use the ref attribute or local state. It is recommended to use the latter approach, because the ref attribute is reserved for only a few use cases. If you want to read about these use cases, I encourage you to read the following article: [When to use Ref on a DOM node in React](https://www.robinwieruch.de/react-ref-attribute-dom-node/).

THe following code snippet is a quick  demonstration on how form state can be used by using the ref attribute. Afterward, I will refactor the example to use the local state which is the best practice anyway.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Search extends React.Component {
  constructor(props) {
    super(props);

    this.onSubmit = this.onSubmit.bind(this);
  }

  onSubmit(event) {
    const { value } = this.input;

    // do something with the search value
    // e.g. propagate it up to the parent component
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

The value from the DOM input node gets retrieved by using the reference to the DOM node. The reference is created by using the ref attribute. Now let's see how to make use of local state to embrace best practices rather than using the reserved ref attribute.

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
    this.onSubmit = this.onSubmit.bind(this);
  }

  onChange(event) {
    const { value } = event.target;

    this.setState({
      query: value
    });
  }

  onSubmit(event) {
    const { query } = this.state;

    // do something with the search value
    // e.g. propagate it up to the parent component
    this.props.onSearch(query);

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

### Controlled Components

The previous example of using form state with local state has one flaw. It doesn't make use of **controlled components**. Naturally a HTML input fields holds its own state. When you enter a value into the input field, the DOM node knows about the value.

However, the value lives in your local state too. You have it in both, the native DOM node state and local state. But you want to make use of a single source of truth. It is a best practice to overwrite the native DOM node state by using the `value` attribute and the value from the local state.

Let's consider again the previous example. The input field had no value attribute assigned. By using the native value attribute and passing the local state as value, you convert an uncontrolled component to a controlled component.

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

### Unidirectional Data Flow

In the previous example, you experienced a typically unidirectional data flow. The Flux architecture, the underlying architecture for several sophisticated state management solutions, coined the term **unidirectional data flow**. You will get to know more about the Flux architecture in a later chapter. But the unidirectional data flow is embraced by local state in React too.

State in React flows only in one direction. State gets updated by using `this.setState()` and is displayed due to the `render()` lifecycle method by accessing `this.state`. Then again, it can be updated via `this.setState()` and the component re-renders.

The previous example, where you have used controlled components, shows the loop of the unidirectional data flow. The input field triggers the `onChange` handler when the input changes. The handler alters the local state. The changed local state triggers an update lifecylce of the component. The update lifecylce runs the `render()` lifecycle method again. The `render()` method makes use of the updated state. The state flows back to the input field to make it a controlled component. The loop is closed. A new loop can be triggered by typing something in the input field again.

The unidirectional data flow makes state management predictable and maintainable. The best practice already spread to other state libraries, view layer libraries and single page application solutions. In the previous generation of SPAs most often other mechanics were used.

For instance, in Angular 1.x you had to use two-way data binding in a model-view-controller (MVC) architecture. That means, once you changed the value in the view, let's say in an input field by typing something, the value got changed in the controller. But it worked vice versa too. Once you changed the value in the controller programmatically, the view, to be more specific the inptu field, displayed the new value.

You might wonder: What's the problem with this approach? Why is everybody using unidirectional data flow instead of bidirectional data flow?

#### Unidirectional vs. Bidirectional Data Flow

React embraces unidirectional data flow. In the past, frameworks like Angular 1.x embraced bidirectional data flow. It was known as two-way data binding. It was one of the reasons that made Angular popular in the first place. But it failed in this particular area too. Especially, in my opinion, this one flaw led a lot of people switch to React. But at this point I don't want to get too opinionated. Why did the bidirectional data flow fail? Why is everyone adopting the unidirection data flow?

The three advantages in unidirectional data flow over bidirectional data flow are prediciablity, maintainability and performance.

**Predicability**: In a scaling application state management needs to stay predicatle. When you alter your state, it should be clear which components care about it. It should be also clear who alters the state in the first place. In an unidirectional data flow one stakeholder alters the state, the state gets stored, and the state flows down from one place, for instance a stateful component, to all child components that are interested in the state.

**Maintainability:** When collaborating in a team on a scaling application, it is a requirement that the state management is predicatblte. Humans are not capable to keep track of bidirectional data flow. It is a limitation by nature. That's why the state management can only stay predictable when it stays maintainable. Otherwise, when people cannot reason about the state, they introduce unefficient state handling.

But maintainability doesn't come without any cost in an unidirectinal data flow. Even though the state is predictable, it needs to be often refactored thoughtfully. In a later chapter you will read about those refactroings such as lifting state or higher order components for local state.

**Performance:** In an unidirectional data flow, the state flows down the component tree. All components that depend on the state have the chance to re-render. Contrary in an bidirectional data flow, it is not always clear who has to update. The state flows in too many directions. The model layer depends on the view layer and the view layer depends on the model layer. It's a vice versa dependency that leads to performance issues in the update lifecycle.
