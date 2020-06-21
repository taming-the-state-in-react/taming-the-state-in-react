## Definitions

Before we dive into state management, this chapter gives you general and state managements definitions to build up common vocabularies for state management. It should help you to read this book effortlessly without leaving space for confusion.

### Pure Functions

"Pure functions" is a concept from the functional programming paradigm. A pure function always returns the same output if given the same input. There is no layer in between that alter the output on the way when the input doesn't change. The layer in between, that could alter the output, is called **side-effect**. Thus, pure functions have no side-effects. Two major benefits of these pure functions are predictability and testability.

### Immutability

Immutability is a concept of functional programming as well. It states that, a data structure is immutable if it cannot be changed. When there is a need to modify the immutable data structure, for instance, an object, you would always return a new object. Instead of altering the object at hand, you would create a new object based on the old object where as the old and new object would have their own instances.

Predictability is one of the benefits of immutable data structures. For instance, when an object is shared through the whole application, any direct changes made to it could lead to bugs because every stakeholder has a reference to this potentially altered object. What happens when an object changes would be unpredictable whereas stakeholders such as UI components depend on this object. In a growing application, it is difficult to oversee the places where its reference uses the object.

Note: The antagonist of immutability is called mutability. It says that an object can be modified.

### State

State is a broad word in modern applications. **Application state** could be anything that needs to be stored and **managed (created, updated, deleted)** in the application. The state can be **remote data** which is fetched from a back-end application or **view data** which lives only on the client-side of the application.

I will refer to the former one as **entity state** and to the latter one as **view state**. Entity state is data retrieved from a back-end application. It could be a list of authors or the user object describing the user that is currently logged in to the application. View state, on the other hand, doesn't need to be stored in the back-end. It is used when you open up a modal or switch a box from preview to edit mode.

Managing the state means creating, updating and deleting state and this will be coined under the umbrella term of state management. State management is a much broader topic. While the mentioned actions are low-level operations, almost implementation details, the architecture, best practices and patterns around state management stay abstract. **State management** involves all these topics to keep your application state durable.

#### The Size of State

State can be an atomic object or one large aggregated object. When speaking about the view state, that only determines whether a pop-up is open or closed, it is an **atomic state object**. When the whole application state can be derived from one aggregated object, which includes all the atomic state objects, it is called a **global state object**. Often, a global state object implies that it is accessible from everywhere.

The state itself can be differentiated into **local state** and **sophisticated state**. The management of this state is called **local state management** and **sophisticated state management**.

#### Local State

The term "local state" is widely accepted on the web development community. Another term might be **internal component state**. Local state is bound to components or component hierarchies. It lives in the view layer. It is not stored somewhere else outside of this view layer. That's why it is called local state because it is co-located to the component.

In React, the local state is embraced by using `this.state` and `this.setState()`. But it can have different implementation and usage in other view layer or SPA solutions. The book explains and showcases the local state in React before diving into sophisticated state management with external libraries such as Redux and MobX.

#### Sophisticated State

I cannot say it is widely termed as "sophisticated state" on the web development community. However, at some point, you need a term to distinguish it from "local state". That's why I often refer to it as a sophisticated state. In other resources, you might find it referred to as **external state**, because it lives outside the UI components or outside the view layer.

Generally, sophisticated state is outsourced to libraries that are library or framework agnostic and thus agnostic to the view layer. But most often they provide a bridge to access and modify state from the view layer. When using only local state in a scaling application, you will allocate too much state along with your components in the view layer. However, at some point, you want to separate view layer and state layer because the state becomes too complex. That's when sophisticated state comes into play.

Two libraries that are known for managing sophisticated state are called Redux and MobX. Both libraries will be explained, discussed and showcased in this book.

#### Visibility of State

Since local state is only bound to the component instance, only the component itself is aware of these properties being state. However, the component can share the state with its child components. In React, the child components are unaware of these properties being state. They only receive these properties as props.

On the other hand, sophisticated state is often globally accessible. In theory, the state can be accessed by each component. Often, it is not the best practice to give every component access to the global state; thus it is up to the developer to bridge only selected components to the global state object. All other components stay unaware of the state and only receive properties as props to act on them.
