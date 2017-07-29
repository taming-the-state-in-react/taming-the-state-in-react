# Redux

Redux is one of the libraries that helps you implementing sophistaicated state management in your applications. It goes beyond the local state. It is one of the solutions you would take in a scaling application in order to tame the state. A React application is a perfect fit for Redux, yet other libraries and frameworks highly adopted it as well.

**Why is Redux that popular in the JavaScript community?** In order to answer that question, I have to go a bit into the past of JavaScript applications. In the beginning, there was one library to rule them all: jQuery. It was mainly used to manipulate the DOM, to amaze with animations and to implement reusbale widgets. It was the number one library in JavaScript. There was no way around it. However, the usage of jQuery skyrocketed and applications grew in size. But not in size of HTML and CSS. It was the size of code in JavaScript. Eventually the code in those applications became a mess, because there was no proper architecture around it. The infamous spaghetti code became a problem in JavaScript applications.

It was about time for noveau solutions to emerge which would go beyond jQuery. These libraries, most of them frameworks, would bring the tools for proper architectures in frontend applications. In addition, they would bring opinionated approaches to solve problems. These solutions enabled developers to implement single page applications (SPAs).

Single page applications became popular when the first generation of frameworks and libraries, among them Angular 1, Ember and Backbone, were released. Suddenly developers had frameworks to build scaling frontend applications. However, as history repeats itself, with every new technology there will be new problems. In SPAs every solution had a different approach for state management. For instance, Angular 1 used the famous two-way data binding. It embraced a bidirectional data flow. Only after applications scaled, the problem in state management became widely known.

During that time React was released by Facebook. It was among the second generation of SPA solutions. Compared to the first generation, it was a library that only leveraged the view layer. It came with an own state management solution though: local state management.

In React the principle of the unidirectional data flow became popular. State management should be more predictable in order to reason about it. Yet, the local state management wasn't sufficient at some point. React applications scaled very well, but ran into the same problems of predictable and maintainable state management. Even though the problems weren't that destructive than in bidirectional data flow applications, there was still a scaling problem. That was the time when Facebook introduced the Flux architecture.

The Flux architecture is a pattern to deal with state management in scaling applications. The official website says that *"[a] unidirectional data flow is central to the Flux pattern [...]"*. The data flows only in one direction. Apart from the unidirectional data flow, the Flux architecture came with 4 essential components: Action, Dispatcher, Store and View. The View is basically the component tree in a modern application. An user can interact with the View in order to trigger an Action. An Action would encapsulate all the necessary information to update the state in the Store(s). The Dispatcher on the way delegates the Actions to the Store(s). The updated state would be propagated to the Views again to update them.

The unidirectional goes in one direction. A View can trigger an Action, that goes through the Dispatcher and Store, and would change the View eventually when the state in the Store changed. The unidirecitonal data flow is enenclosed in this loop. Then again, a view can trigger another action. Since Facebook introduced the Flux architecture, the View was associated with React.

You can [read more about the Flux architecture](https://facebook.github.io/flux/) on the official website. There you will find a [video about its introduction at a conference](https://youtu.be/nYkdrAPrdcw?list=PLb0IAmt7-GS188xDYE-u1ShQmFFGbrk0v) too. If you are interested about the origins of Redux, I highly recommend to read and watch the material.

After all, Redux became the successor library of the Flux architecture. Even though there were a bunch of solutions around the Flux architecture, Redux managed to succeed. But why did it succeed?

[Dan Abramov](https://twitter.com/dan_abramov) and [Andrew Clark](https://twitter.com/acdlite) are the creators of Redux. It was [introduced by Dan Abramov at React Europe](https://www.youtube.com/watch?v=xsSnOQynTHs) in 2015. However, the talk by Dan doesn't introduce Redux per se. Instead the talk introduced a problem that Dan Abramov faced that led to Redux. I don't want to foreclose the content of the talk, that's why I encourage you to watch the video. If you are keen to learn Redux, you should dive into the problem that was solved by it for Dan Abramov.

Nevertheless, one year later, again at React Europe, Dan Abramov reflects about the journey of Redux and its success. He mentions a few things that made Redux successful in his opinion. The two main take aways of the success of Redux were: the problem and the constraints.

Redux was developed to solve a problem. The problem was explained by Dan Abramov one year ealier when he introduced Redux. It was not yet another library. It was a library that solved a problem. Time Traveling and Hot Reloading were the stress test for Redux.

The second key take away, the contrainss, were another key factor to its success. Redux managed to shield away the problem with a simple API and a thoughtful way to solve the problem of state management itself.

[You can watch the talk](https://www.youtube.com/watch?v=uvAXVMwHJXU) too. However, maybe it makes sense to watch it after the book introduced you the basics of Redux.

- TODO http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-1/

- TODO http://blog.isquaredsoftware.com/2017/05/idiomatic-redux-tao-of-redux-part-2/

- TODO introduction: http://blog.isquaredsoftware.com/presentations/2017-02-react-redux-intro/#/58 or check if it deals with react-redux
