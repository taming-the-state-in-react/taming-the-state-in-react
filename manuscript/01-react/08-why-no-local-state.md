## The Lies of Local State Management

State management is a controversial topic. You will find a ton of discussion and opinions around it. You will find it as reoccuring topic not only in React, but also in other solutions for modern applications. The book is my attempt to give you consistency for these opinions and enable you to learn state management step by step.

The following statement is controversial: *The local state in React is sufficient for most of your application. You will not need a sophisticated state managament solutions like Redux or MobX.*

Personally I agree with the statement. You can build quite large applications with local state only. You should be aware of best practices and patterns to scale it, but it is doable. You can spare a lot of application complextity by using local state only. Once your application scales, you might want to apply sophisticaed state management.

The next statement might be controversial too: *Once you have a sophisitcated state management in place, you shouldn't use local state anymore.*

Personally I disagree with the statement. Not every state should live in a sophisticated state management. There are use cases where local state is applicable in large applications. Especially when considering entity state and view state: The view state can most often live in a local state, because it is not shared widely across the application. But the entity state can live in a sophisticated state, because it is shared across multiple components. It might need to be accessible and modifyable by multiple components across your application.

Last but not least, another controversial statement: *You don't need local state, learn Redux instead when you learn React.*

Personally I strongly disagree with the statement. If you want to develop applications with React, you should certainly be aware of local state in React. You should have build applications with it before you start to learn sophisticated state management solutions like Redux. You need to run into local state management problems before you get the help of sophisticated state management solutions.

These were only three controversial statemanemts. But there are way more opinions around the topic. In the end, you should make your own experiences to get to know what makes sense for you.

## The Flaw of Local State Management

In order to come to a conclusion of local state management there is one open question: What's the problem with local state management? Developers wonder why they need sophisticated state management in order to tame their state. In other scenarios people never wonder about it, because they learned sophistiacted state management from the beginnign without using local state. That might be not the best approahc in the first place, because you have to experience a problem before you use a solution for it. You can't skip the problem and use the solution right away.

So what's the problem in local state management? It doesn't scale in large applications. It doesn't scale implementation wise, but it might doesn't scale in a team of developers too.

Implementaiton wise it doesn't scale because too many components across your application share state. They need to access the state, need to modify it or need to remove it. In a small applications these components are not far away from each other. You can apply best practices like lifting state up and down to keep the state management clean. At some point components are too far away from each other. The state needs to be lifted up the component tree all the way up. Still, child components could be multiple levels below the stateful component. The state would creep through all components in between even though these component don't need access to it. It becomes a state soup.

Local state can become unmaintanable. It is already difficult for one person to keep the places in mind where local state is used in the component tree. When a team of developers implements one application, it becomes even more difficult to keep track of it. Usually it is not neccessary to keep track about the local state. In a perfect world, everyone would lift state up and down to keep it clean. In the real world, code doesn't get refactored as often as it should be. The state creeps through all component even though they don't need it.

One could argue that the difficult maintainablity applies for sophisticaed state as well. That's true, there are pitfalls again that people need to avoid to keep the state management maintainable. But at least the state management is gathered at one place to maintain it. It doesn't get too mixed up with the view layer. There are only bridges that connect the view with the state.