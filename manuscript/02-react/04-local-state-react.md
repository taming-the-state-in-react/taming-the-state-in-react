## Local State in React

The book uses React as view layer for demonstrating the local state in a web application. The following chapter focusses on the local state in React before it dives into sophisticated state management with Redux and MobX. As mentioned, the concept of local state should be known in other SPA solutions, too, and thus be applicable in those solutions.

So, what does local state look like in a React component?

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

The example shows a `Counter` component that has a `counter` property in the local state object. It is defined with a value of `0` when the component gets instantiated by its constructor. In addition, the `counter` property from the local state object is used in the render method of the component to display its current value.

There is no state manipulation in place yet. Before you start to manipulate your state, you should know that you are never allowed to mutate the state directly: `this.state.counter = 1`. That would be a direct mutation. Instead, you have to use the React component API to change the state explicitly by using the `this.setState()` method. It keeps the state object immutable, because the state object isn't changed but a new modified copy of it is created.

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

Now, the button `onClick` handler should invoke the class methods to alter the state by either incrementing or decrementing the counter value. Then, the update functionality with `this.setState()` is performing a **shallow merge** of objects. What does a shallow merge mean? Imagine you had the following state in your component, two arrays with objects:

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.state = {
  authors: [...],
  articles: [...],
};
~~~~~~~~

When updating the state only partly, for instance the authors, the other part, in this case the articles, are left intact.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState({
  authors: [
    { name: 'Robin', id: '1' }
  ]
});
~~~~~~~~

It only updates the `authors` array with a new array without touching the `articles` array. That's called a shallow merge. It simplifies the local state management for you so that you don't have to keep an eye on all properties at once in the local state.

### Stateful and Stateless Components

Local state can only be used in React ES6 class components. The component becomes a **stateful component** when state is used. Otherwise, it can be called **stateless component** even though it is still a React ES6 class component. This can be the case if you still need to use React's lifecycle methods.

On the other hand, **functional stateless components** have no state, because, as the name implies, they are only functions and thus, they are stateless. They get input as props and return output as JSX. In a stateless component, state can only be passed as props from a parent component. However, the functional stateless component is unaware of the props being state in the parent component. In addition, callback functions can be passed down to the functional stateless component to have an indirect way of altering the state in the parent component again. A functional stateless component for the Counter example could look like the following:

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

Now only the props from the parent component would be used in this functional stateless component. The `counter` prop would be displayed and the two callback functions, `onIncrement()` and `onDecrement()` would be used for the buttons. However, the functional stateless component is not aware whether the passed properties are state, props or some other derived properties. The origin of the props doesn't need to be in the parent component after all, it could be somewhere higher up the component tree. The parent component would only pass the properties or derived properties along the way. In addition, the component is unaware of what the callback functions are doing. It doesn't know that these alter the local state of the parent component.

After all, the callback functions in the stateless component would make it possible to alter the state somewhere above in one of the parent components. Once the state was manipulated, the new state flows down as props into the child component again. The new `counter` prop would be displayed correctly, because the render method of the child component runs again with the incoming changed props.

The example shows how local state can traverse down from one component to the component tree. To make the example with the functional stateless component complete, let's quickly show what a potential parent component, that manages the local state, would look like. It is a React ES6 class component in order to be stateful.

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

It is not by accident that the suffixes in the naming of both `Counter` components are `Container` and `Presenter`. It is called the [container and presentational component pattern](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). It is most often applied in React, but could live in other component centred libraries and frameworks, too. If you have never heard about it, I recommend reading the referenced article. It is a widely used pattern, where the container component deals with "How things work" and the presenter component deals with "How things look". In this case, the container component cares about the state while the presenter component only displays the counter value and provides a handful of click handler yet without knowing that these click handlers manipulate the state. Note that the presenter component is called presentational component in the referenced article. I shortened the name from presentational to presenter component for the sake of convencience.

Container components are the ideal candidates to manage state while the presenter components only display it and act on callback functions. You will encounter these container components more often in the book, when dealing with the concepts of higher-order components, that could potentially manage local state, and connected components.

### Props vs. State

The previous example made clear that there is a difference between state and props in React. When properties are passed to a child component, whether it is state, props or derived properties, the child component isn't aware of the kind of properties. It sees the incoming properties as props. That's perfect, because the component shouldn't care at all about the kind of properties. It should only make use of them as simple props.

The props come from a parent component. In the parent component these props can be state, props or derived properties. It depends on the parent component, if it manages the properties itself (state), if it gets the properties from a parent component itself (props) or if it derives new properties from the incoming props coming from its parent component along the way (derived properties).

After all, you can't modify props. Props are only properties passed from a parent component to a child component. On the other hand, the local state lives in the component itself. You can access it by using `this.state`, modify it by using `this.setState()`, and pass it down as props to child components.

When one of these objects changes, whether it is the props that come from the parent component or the state in the component, the update lifecycle methods of the component will run. One of these lifecycle methods is the `render()` method that updates your component instance based on the props and state. The correct values will be used and displayed after the update ran in your component.

When you start to use React, it might be difficult to identify props and state. Personally, I like the [rules in the official React documentation](https://reactjs.org/docs/thinking-in-react.html) to identify state and props:

* Are the properties passed from the parent component? If yes, the likelihood is high that they aren't state. Though it is possible to save props as state, there are little use cases. It should be avoided to save props as state. Use them as props as they are.

* Are the properties unchanged over time? If yes, they don't need to be stateful, because they don't get modified.

* Are the properties derivable from local state or props? If yes, you don't need them as state, because you can derive them. If you allocated extra state, the state has to be managed and can get out of sync when you miss to derive the new properties at some point.

### Form State

A common use case in applications is to use HTML forms. For instance, you might need to retrieve user information like a name or credit card number or submit a search query to an external API. Forms are used everywhere in web applications.

There are two ways to use forms in React. You can use the ref attribute or local state. It is recommended to use the local state approach, because the ref attribute is reserved for only a few use cases. If you want to read about these use cases when using the ref attribute, I encourage you to read the following article: [When to use Ref on a DOM node in React](https://www.robinwieruch.de/react-ref-attribute-dom-node/).

The following code snippet is a quick demonstration on how form state can be used by using the ref attribute. Afterward, the code snippet will get refactored to use the local state which is the best practice anyway.

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

The value from the input node is retrieved by using the reference to the DOM node. It happens in the `onSubmit()` method. The reference is created by using the ref attribute in the `render()` method.

Now let's see how to make use of local state to embrace best practices rather than using the reserved ref attribute.

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

You don't need to make use of the ref attribute anymore. You can solve the problem by using local state only. The example demonstrates it with only one input field yet it can be used with multiple input fields, too. You would only need to allocate more properties, one for each input field, in the local state.

### Controlled Components

The previous example of using form state with local state has one flaw. It doesn't make use of **controlled components**. Naturally, a HTML input field holds its own state. When you enter a value into the input field, the DOM node knows about the value. That's the native behavior of HTML elements, otherwise they wouldn't work on their own.

However, the value lives in your local state, too. You have it in both, the native DOM node state and local state. But you want to make use of a single source of truth. It is a best practice to overwrite the native DOM node state by using the `value` attribute on the HTML element and the value from the local state from the React component.

Let's consider the previous example again. The input field had no value attribute assigned. By using the native value attribute and passing the local state as value, you convert an uncontrolled component to a controlled component.

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

Now the value comes from the local state as single source of truth. It cannot get out of sync with the native DOM node state. This way, you can provide an initial state for the DOM node state too. Otherwise, try to have an initial local state for the `query` in your local state, but don't provide the `value` attribute to the input field. Your state would be out of sync in the beginning, because the input field would be empty even though the local state of the React component says something else.

### Unidirectional Data Flow

In the previous example, you experienced a typical unidirectional data flow. The Flux architecture, the underlying architecture for several sophisticated state management solutions such as Redux, coined the term **unidirectional data flow**. You will get to know more about the Flux architecture in a later chapter. But the essence of an unidirectional data flow is embraced by local state in React, too. State in React flows only in one direction. State gets updated by using `this.setState()` and is displayed due to the `render()` lifecycle method by accessing `this.state`. Then again, it can be updated via `this.setState()` and a component re-renders.

The previous example, where you have used controlled components, shows the perfect loop of the unidirectional data flow. The input field triggers the `onChange` handler when the input changes. The handler alters the local state. The changed local state triggers an update lifecycle of the component. The update lifecycle runs the `render()` lifecycle method again. The `render()` method makes use of the updated state. The state flows back to the input field to make it a controlled component. The loop is closed. A new loop can be triggered by typing something into the input field again.

The unidirectional data flow makes state management predictable and maintainable. The best practice already spread to other state libraries, view layer libraries and SPA solutions. In the previous generation of SPAs, most often other mechanics were used. For instance, in Angular 1.x you would have used two-way data binding in a model-view-controller (MVC) architecture. That means, once you changed the value in the view, let's say in an input field by typing something, the value got changed in the controller. But it worked vice versa, too. Once you had changed the value in the controller programmatically, the view, to be more specific the input field, displayed the new value. You might wonder: What's the problem with this approach? Why is everybody using unidirectional data flow instead of bidirectional data flow now?

#### Unidirectional vs. Bidirectional Data Flow

React embraces unidirectional data flow. In the past, frameworks like Angular 1.x embraced bidirectional data flow. It was known as two-way data binding. It was one of the reasons that made Angular popular in the first place. But it failed in this particular area, too. Especially, in my opinion, this particular flaw led a lot of people to switch to React. But at this point I don't want to get too opinionated. So why did the bidirectional data flow fail? Why is everyone adopting the unidirectional data flow?

The three advantages in unidirectional data flow over bidirectional data flow are predicability, maintainability and performance.

**Predicability**: In a scaling application, state management needs to stay predictable. When you alter your state, it should be clear which components care about it. It should also be clear who alters the state in the first place. In an unidirectional data flow one stakeholder alters the state, the state gets stored, and the state flows down from one place, for instance a stateful component, to all child components that are interested in the state.

**Maintainability:** When collaborating in a team on a scaling application, one requirement of state management is predictability. Humans are not capable to keep track of a growing bidirectional data flow. It is a limitation by nature. That's why the state management stays more maintainable when it is predictable. Otherwise, when people cannot reason about the state, they introduce inefficient state handling. But maintainability doesn't come without any cost in a unidirectional data flow. Even though the state is predictable, it often needs to be refactored thoughtfully. In a later chapter, you will read about those refactorings such as lifting state or higher-order components for local state.

**Performance:** In a unidirectional data flow, the state flows down the component tree. All components that depend on the state have the chance to re-render. Contrary to a bidirectional data flow, it is not always clear who has to update according to state changes. The state flows in too many directions. The model layer depends on the view layer and the view layer depends on the model layer. It's a vice versa dependency that leads to performance issues in the update lifecycle.

These three advantages show the benefits of using a unidirectional data flow over an bidirectional data flow. That's why so many state management and SPA solutions thrive for the former one nowadays.
