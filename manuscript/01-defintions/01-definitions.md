# Definitions

This chapter gives you general defintions and defintions for state managements to build up on a common vocabulary for state management for this book. It should help you to follow the book effortless when reading it without leaving space for confusion.

## Pure Functions

Pure functions is a concept from the functional programming paradigm. It says that a pure function always returns the same output when giving it the same input. There is no layer in between that could alter the output when the input stays the same. The layer in between, that could possibly alter the output, is called **side-effect**. Thus pure functions have no side-effects. Two major benefits of these pure functions are predictablity and testability.

## Immuatbiltiy

Immutabiltiy is a concept from functional programming too. It says that a data structure is immutable when it is never changed. When there is the need to modify the immutable data structure, for instance an object, you would always return a new object. Rather than altering the object at hand, you would create a new modified object based on the old object and the modification. The old and new object would have their own instances.

Immutable data structures have the beneift of predictability. For instance, when sharing an object through the whole application, it could lead to bugs when altering the object directly, because every stakeholder has a reference to this altered object. It would be unpredictable what happens when an object changes an a handful of stakeholders are dependent on this object. In a growing application it is difficult to oversee the places where the object is currently used.

The antagonsit of immutablity is called mutability. It says that an object can be altered (mutated) directly.

## State

State is a broad word in modern applications. When speaking about application state, it could be anything that needs to live and be modified in the browser. It could be entity state that was retrieved from a backend application or a view state in the application, for instance, when toggling a popup to show additional information.

I will refer to the former one as **entity state** and to the latter one as **view state**. Entitiy state is data retrieved from the backend. It could be a list of authors or the user object describing the user that is currently logged in to the application. View state, on the other hand, doesn't need to be stored in the backend. It is used when you open up a modal or navigate through your application.

When speaking about managing the state, meaning initializing, modifying and deleting state, it will be coined under the umbrella term of state management. Yet state management is a much broader topic. While the mentioned actions are low-level operations, almost implementation details, the architecture, best practices and patterns around state management stay abstract. **State management** involves all these topics to keep your application state durable.

### The Size of State

State can be an atomic object or one large aggregated object. When speaking about the view state, that only determines if a popup is open or closed, it is an **atomic state object**. When the whole application state can be derived from on aggregated object, which includes all the atomic state objects, it is called a **global state object**. Often a global state object implies that it is accessible from everyone, but that's not necessarly the case.

The state itself can be differentiated into **local state** and **sophisticated state**. The management of this state is called **local state management** and **sophisticated state management**.

### Local State

The naming local state is widely accepted in the web development community. Another term might be **internal component state**.

Local state is bound to components. It lives in the view layer. It is not stored somewhere else. That's why it is called local state.

In React the local state is embraced by using `this.state` and `this.setState()`. But it can have a different implementation and usage in other SPA solutions. The book explains and showcases the local state in React before diving into sophisticated state management with external libraries.

### Sophisticated State

I cannot say that it is widely agreed on to call it sophisticated state in the web development community. However, at some point you need a term to distinguish it from local state. That's why I often refer to it as sophisticated state. In other resources, you might find it reffered as **external state**, because it lives outside of the local component or outside of the view layer.

Sophistaicated state is most often outsourced to libraries that are library or framwork agnostic and thus agnostic to the view layer. But most often they provide a bridge to access and alter them from the view layer. When using only local state in a scaling application, you will allocate too much state along your components in the view layer. However, at some point you want to separate these concerns. That's were sophistatced state comes into play.

Two libraries that are known for handling sophistated state are known as Redux and MobX. Both libraries will be explained, discussed and showcased in this book.

### Visibility of State

Since local satte is only bound to the component instance, only the component itself is aware of these properties as state. However, the component can share the state to its child components. In React the child components are unaware of these prioperties being state in their parent component. They only receive these properties as props.

On the other hand, sophisticated state is often globally accessible. In theory the state can be accessed by each component. Often it is no good practice to give every component access to the global state, thus it is up to the developer to bridge only selected components to the global state object. All other components stay unaware of the state and only receive properties to act on.