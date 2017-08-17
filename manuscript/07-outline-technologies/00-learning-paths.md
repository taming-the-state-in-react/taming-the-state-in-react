# Beyond Redux and MobX

So far the book has taught you different approaches of state management. Whether you are using React, an alternative view layer library or a sophisticated SPA library; most of them will come with a built-in solution to deal with local state. The book has shown you React's local state management and demonstrated approaches to scale it in plain React applications. Afterward, you have learned extensively about Redux as sophisticated state management library. It can be used in combination with any view layer or SPA library. The book has taught you how to use it in React applications too. As alternative to Redux, you have read about MobX as sophisticated state management library. It comes with its own advantages and disadvantages. After all, Redux and MobX give you two different approaches to opt-in state management to your application. However, you should never forget about your local state management solution to keep your state coupled to your components rather than exposing it in your global state.

What else could you use for state management in modern JavaScript applications? There is another solution that should be mentioned in the book: GraphQL and **its clients**. [GraphQL](http://graphql.org/) itself hasn't anything to do with state management. It is used on the server-side. Before diving into the state management when using GraphQL on the server-side, the chapter explains shortly GraphQL.

## GraphQL and REST

GraphQL is a query language for your API introduced by Facebook. In addition, it is a server-side runtime for executing queries by using a type system you define for your data. GraphQL isn't tied to any database or storage engine and is instead backed by your existing code and data. Keeping it simple for the sake of this book: GraphQL is an alternative to a [REST API](https://www.robinwieruch.de/what-is-an-api-javascript/) to expose data to client applications.

What's the difference between REST and GraphQL? REST is an architectural style that enables applications to communicate with each other. It enables you to build a client-server architecture where REST defines the API between both entities. REST isn't any technology but a paradigm that makes use of HTTP in an opinionated way. RESTful services expose resources, for instance a list of todos or a single todo item via its unique identifier, as URIs. In addition, HTTP methods (GET, POST, PUT, DELETE, PATCH) are used to access and update these resources.

For instance, a GET method on the following URI could return a JSON object with a list of todo items:

{title="Code Playground",lang="javascript"}
~~~~~~~~
GET https://my-api-domain/todos
~~~~~~~~

In another example, a DELETE method would be used to remove a single todo item by its id:

{title="Code Playground",lang="javascript"}
~~~~~~~~
DELETE https://my-api-domain/todos/a3h96se32
~~~~~~~~

That way a client can communicate with a server to access and update the data. Most often the data is managed in a database on the server-side.

The REST paradigm has its limits though. Imagine you would have a web and a mobile client of your application. Both are consuming data that is fetched from a backend application. Whereas the web client would fetch a rich set of entities from the backend, the mobile client, due to its data volume and display limitations, would only fetch the most important properties of the entities. These two differing requests would already lead to two different endpoints when following the REST paradigm.

When using GraphQL, you can define in your client application which properties you want to fetch for every entity. In a mobile client you would only fetch the most important properties. However, in a web client you may want to fetch all properties of an entity. GraphQL offers you a query language to fetch dynamically data from your API.

## GraphQL Clients

Basically you could fetch data from a GraphQL API the same way as from a REST API. There is no magic yet in your client application. However, there exist GraphQL clients, when exposing GraphQL API from your server, that can be used on top of your client application library/framework to enable state management.
