## Requirements

What are the requirements to read the book? First of all, you should be familiar with the basics of web development. You should know how to use HTML, CSS and JavaScript. Perhaps it makes sense to know the term [API](https://www.robinwieruch.de/what-is-an-api-javascript/) too, because you will use those in the book. But there is more that I want to give you on the way when reading this book.

### React

The book uses React as library to teach modern state management. It is a perfect choice for demonstrating and learning state managament in modern applications. Because React is only a view layer, it is up to you to decide how to deal with the state in your application. The state management layer is exchangeable.

After all, it's not neccessary to be a React developer in order to learn about state management in modern applications. If you are developing with another SPA framework, such as Angular, or view layer library, such as Vue, all these things about state managament taught in this book can still be applied in your appliactions. The state management solutions are framework and library agnostic.

Still, since the book uses React for the sake of teaching state management in a proper context, if you are not familiar with React or need to have a refresher on the topic, I can recommend you to read the precedent book: [The Road to learn React](https://www.robinwieruch.de/the-road-to-learn-react/). It is open source. It should enable everyone to learn React. However, you can decide to pay something to support the project.

Even though the book is open source, people with lacking education have no access to open source in the first place. They have to be educated in the English language to be enabled to access it. The Road to learn React attempts [to support education in the developing world](https://www.robinwieruch.de/giving-back-by-learning-react/) on a occasionally basis, but it is a tough since the book itself is pay what you want.

In addition, the Road to learn React teaches you to make the transition from JavaScript ES5 to JavaScript ES6. After you have read the Road to learn React, you should have all the knowledge to read Taming the State. It builds perfectly up on the React book.

### Editor and Terminal

What about the development environment? You will need a running editor (IDE) and terminal (command line tool). You can [follow my setup guide](https://www.robinwieruch.de/developer-setup/). It is adjusted for Mac users, but you can substitute most of the tools for other operating system. There are a ton of articles out there that will show you to setup a web development environment in a more elaborated way.

Optionally, you can use git and GitHub on your own, while conducting the exercises in the book, to keep your projects and the progress in repositories on GitHub. There exists a [little guide](https://www.robinwieruch.de/git-essential-commands/) on how to use these tools. But once again, it is not mandatory for the book and can be overwhelming when learning everything from scratch.

### Node and NPM

Last but not least, you will need an installation of [node and npm](https://nodejs.org/en/). Both are used to manage libraries you will need along the way. In this book, you will install external node packages via npm (node package manager). These node packages can be libraries or whole frameworks.

You can verify your versions of node and npm on the command line. If you don't get any output in the terminal, you need to install node and npm first. These are only my versions during the time writing this book:

{title="Command Line",lang="text"}
~~~~~~~~
node --version
*v7.4.0
npm --version
*v4.0.5
~~~~~~~~

When you read the Road to learn React, you should be familiar with the setup already. The book gives you a short introduction into the npm ecosystem on the command line too.