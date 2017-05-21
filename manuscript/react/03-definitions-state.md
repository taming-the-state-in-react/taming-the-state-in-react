# Defitnions in State Management

In the beginning, I want to give you defintions to state managament in modern applications. With these definitions at your disposal, I want to avoid that you get confused while reading the book. They should clearly seperate the different domains.

## State

State is a broad word in modern web applications. When speaking about application state, it could be anything that needs to live and be modified in the browser. It could be data that was retrieved from a backend application or a view state in the application, for instance, when toggling a popup to show additional information.

Sometimes I will refer to the former one as **entity state** and to the latter one as **view state**. Entitiy state is data retrieved from the backend. It could be a list of authors or the user object describing the user that is currently logged in to the application. View state, on the other hand, doesn't need to be stored in the backend. It is used when you open up a modal or navigate through your application.

When speaking about managing the state, meaning initializing, modifying and deleting state, it will be coined under the umbrella term of state management. Yet state management is a much broader topic. While the mentioned actions are low-level operations, almost implementation details, the architecture, best practices and patterns around state management stay abstract. **State management** invoves all these topics to keep your application state consistent.

## The Size of State

State can be an atomic object or one large aggregated object. When speaking about the view state that only determines if a popup is open or closed, it is an **atomic state object**. When the whole application state can be derived from on aggregated object, which includes all the atomic state objects, it is called a **global state object**. A global state object most often implies that it is accessible from everyone.

For instance, in games most often the whole application state, the global state object, is called a game object. You can derive the whole state for your game from it. It can be the position of your character in a role play game but also your inevntory of items of your character. In addition it could be the positions of your enemies approaching your character. Imagine that you only need to load the application itself and use one global state object to derive everything you need for your application. Later on, you will learn about it as dehydration and rehydration of state. In adition, you will get to know how this can help for server-side rendering. (TODO check if you really do it!)

The state itself can be differentiated into **local state** and **sophisticated state**. The management of this state is called **local state management** and **sophisticated state management**.

## Changing the State

- state is never final
- it can be changed, modified, altered, manipulated
- the change happens in mutable or immutable ways, more general it could be divided into functional or non functional approaches

## Local State

The naming local state is widely accepted in the web development community.

Other terms are:

* internal component state

Local state is bound to components. It lives in the view layer. It is not stored somewhere else. That's why it is called local state.

In React the local state is embraced by using `this.state` and `this.setState()`. But it can have a different implementation and usage in other SPA solutions. The book explains and showcases the local state in React before diving into sophisticated state management.

## Sophisticated State

I cannot say that it is widely agreed on to call it sophisticated state in the web development community. However, at some point you need a term to distinguish it from local state. That's why I often refer to it as sophisticated state. In other resources you might find it reffered as **external state**, because it lives outside of the local component or outside of the view layer.

Sophistaicated state is most often outsourced to libraries that are indepnendent to the view layer. But most often they provide a connection to access and alter them from the view layer. When using only local state in a scaling application, you will allocate too much state along your components in the view layer. However, at some point you want to separate these concerns. That's were sophistatced state comes into play.

Two libraries that are known for handling sophistated state are known as Redux and MobX. Both libraries will be explained, discussed and showcased in this book.

## Visibility of State

- local only bound to the component instance and child components in the view layer
- sophistaticated state is most often globally accesible

- because of that the responsibilites are coarse grained clear: view state belongs into the local state and entitity state belongs into the global state obect
