# (React in) Redux FAQ

I intend to grow this section organically to answer frequently asked questions to the best of my knowledge. These are questions that come up often when discussing Redux standalone or as complementing part of React.

## Redux vs. Local State

When introducing Redux to a React application, people are unsure how to treat the local state with `this.state` and `this.setState()`. Should they replace the local state entirely with Redux or keep a mix of both? The larger part of the community would argue that it is the latter and I agree with it. Local state doesn't become obsolete when using a sophisticated state management library such as Redux. You would still use it.

Imagine your application grows in lines of code and in number of developers working on this application. Your global state in Redux will necessarily grow, too. However, you want to keep the global state meaningful and reusable from multiple parties (developers, components) in your application. That's why not everything should end up in the global state. In a growing application, you should always revisit your global state and make sure that it is not cluttered and arranged thoughtfully.

The cluttering happens when too much state ends up in the global state that is only used by a single party (one component, one part of the component tree). You should think twice about this kind of state and evaluate whether it would make more sense to put in the local state. Always ask yourself: Who is interested in this state? A balanced mixture of local state and sophisticated state will make your application maintainable and predictable in the long run.

The boundaries between local state and sophisticated state will blur when using a state management library like MobX as alternative to Redux. You will learn about MobX later in this book. But there too, you can plan your state thoughtfully in advance in your application.

In general, the usage of Redux state should be kept to a minimum. A good rule of thumb is to keep the state close to your component with local state but evaluate later whether another party is interested in the state. If another party manages an equivalent state structure in its local state, you could use a reusable higher-order component that manages the state. If the state is shared, you could try to lift your state up or down the component hierarchy. However, if lifting state doesn't solve the problem for you, because the state is shared across the application, you should consider to use Redux for it. In the end, after revisiting all your possibilities when only using React's local state, you might not need Redux in your application.

## View vs. Entity State

In the beginning of the book, I differentiated between view state and entity state. Imagine you have a page that does both displaying a list of items and showing opt-in modals for each item to remove or edit the item. While the former would be the entity state, the latter would be the view state. The entity state, the list of items, most often comes from a backend application. But the view state is triggered by the user only in the frontend. Is there a pattern of where to store which kind of state?

The entity state often comes from the backend. It is fetched asynchronously. In applications, you want to avoid to fetch entities more than once. A good practice would be to fetch the entities once and not again when they are already there. Thus, they would have to be stored somewhere where several parties know that these entities are already fetched. The global state is a perfect place to store them.

The view state is altered only in the frontend. Often, it isn't shared across the application, because, for instance, only the modal knows if it is opened or closed. Since it doesn't need to be shared, you can use the local state and avoid to clutter in the global state.

Imagine you have a component that has tabs. Each tab gives you the possibility to change the representation of displayed items. For instance, the user can choose a grid or a list layout ti display items. It is absolutely fine to store this state in the local state. In addition, you can give your user an improved user experience. You could store the selected tab in the local storage of your browser, too, and when the user returns to the page, you rehydrate the state from the local storage into your local state. The user will always find their preferred tab as the selected one.

## Accidental vs. Planned State

Can you plan your state? I would argue that you can plan it. You can plan which part of the state goes into the global state and which part goes into the local state. For instance, you know about the view and entity states. In addition, you can put some of your state in the local storage to improve the user experience. However, in an evolving application, your state grows and the structure changes. What can you do about it? My recommendation is always revisiting your state arrangements and structure. There is always room for improvements. You should refactor it early to keep it maintainable and predictable in the long run.
