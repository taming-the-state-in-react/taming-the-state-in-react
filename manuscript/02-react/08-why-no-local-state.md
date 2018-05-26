## Transition to Sophisticated State

By now you have learned about the basics in local state management and how to scale it in a growing application with a variety of techniques and best practices. These learnings should give you a great surface area to apply them in your applications to scale state management without using a sophisticated state management library and by only using local state. So what's next in this book? The following chapters will give you a transition from local state management to sophisticated state management.

### The Controversies of Local State Management

State management is a controversial topic. You will find a ton of discussions and opinions around it. You will find it as recurring topic not only in React, but also in other SPA or view layer solutions for modern web applications. The book is my attempt to give you consistency for these opinions and enable you to learn state management step by step.

The following statement is controversial: *The local state in React is sufficient for most of your application. You will not need a sophisticated state management solutions like Redux or MobX.*

Personally, I agree with the statement. You can build quite large applications with only local state management. You should be aware of best practices and patterns to scale it, as you have learned them in the previous chapters, but it is doable. You can spare a lot of application complexity by using plain local state. Once your application scales, you might want to consider using a sophisticated state management library at some point.

The next statement might be controversial, too: *Once you have a sophisticated state management library in place, you shouldn't use local state anymore.*

Personally, I strongly disagree with the statement. Not every state should live in a sophisticated state management. There are many use cases when local state is applicable in large applications. Especially when considering entity state and view state: The view state can most often live in a local state, because it is not shared widely across the application. But the entity state most likely lives in a sophisticated state, because it is shared across multiple components. It might need to be accessible and modifiable by multiple components across your application.

Last but not least, another controversial statement: *You don't need React's local state, you can learn Redux instead altogether with React.*

I strongly disagree with the statement, too. If you want to develop applications with React, you should certainly be aware of local state in React. You should have built applications with it before you start to learn and apply sophisticated state management solutions. You need to run into local state management problems before you get the help of sophisticated state management solutions. You will always need local state management, even for large applications.

These were only three controversial statements. But there are way more opinions around the topic. In the end, you should make your own experiences to get to know what makes sense to you. However, if you are transitioning from plain React to React + Redux or React + Redux, I highly recommend reading this article before continuing reading the book: [Things to learn in React before using Redux](https://www.robinwieruch.de/learn-react-before-using-redux/).

### The Flaw of Local State Management

In order to come to a conclusion of local state management, there is one open question: What's the problem with local state management? Developers wonder why they need sophisticated state management in order to tame their state. In other scenarios, people never wonder about it, because they have learned sophisticated state management from the beginning without using local state. That might not be the best approach in the first place, because you have to experience a problem before you use a solution for it. You can't skip the problem and use the solution right away.

So what's the problem with using only local state management? It doesn't scale in large applications. It doesn't scale implementation-wise, but it probably doesn't scale in a team of developers too.

Implementation-wise it doesn't scale because too many components across your application share state. They need to access the state, need to modify it or need to remove it. In a small application, these components are not far away from each other. You can apply best practices like lifting state up and down to keep the state management maintainable. At some point, components are too far away from each other. The state needs to be lifted the component tree all the way up. Still, child components could be multiple levels below the stateful component. The state would creep through all components in between even though these components don't need access to it. That's there you may want to consider React's context API, but it shouldn't replace your whole state management architecture.

Local state can become unmaintainable. It is already difficult for one person to keep the places in mind where local state is used in the component tree. When a team of developers implements one application, it becomes even more difficult to keep track of it. Usually it is not necessary to keep track about the local state. In a perfect world, everyone would lift state up and down to keep it maintainable. In the real world, code doesn't get refactored as often as it should be. The state creeps through all components even though they don't consume it.

One could argue that the issues of maintainability apply for sophisticated state as well. That's true, there are pitfalls again that people need to avoid to keep the state management maintainable. For instance, not every component should know about the global state. However, at least the state management is gathered at one place, in a global state, to maintain it. It doesn't get too mixed up with the view layer. There are only bridges that connect the view layer with the state layer. Thus it is a wise decision to apply sophisticated state management in larger applications in order to tame the state. So let's get into it!