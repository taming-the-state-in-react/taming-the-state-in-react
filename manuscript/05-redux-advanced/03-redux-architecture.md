# Redux State as Architecture

The book taught you the practical usage of Redux. You have learned about the main parts in the Redux state management architecture: actions, reducers and the store.

{title="Concept Playground",lang="text"}
~~~~~~~~
Action -> Reducer(s) -> Store
~~~~~~~~

The chain is connected to the view layer by something (e.g. react-redux with `mapStateToProps()` and `mapDispatchToProps()`) that enables you to connect your components to the state. These components have access to the Redux store. They are used to receive state or to alter the state. They are a specialized case of a container component in the presenter and container pattern when using components.

{title="Concept Playground",lang="text"}
~~~~~~~~
View -> (mapDispatchToProps) -> Action -> Reducer(s) -> Store -> (mapStateToProps) -> View
~~~~~~~~

All other components are not aware of any local or sophisticated state management solution. They only receive props, except they have their own local state management (such as `this.state` and `this.setState()` in React).

{title="Concept Playground",lang="text"}
~~~~~~~~
View -> Connected View (mapDispatchToProps) -> Action -> Reducer(s) -> Store -> Connected View (mapStateToProps) -> View
~~~~~~~~

State can be received directly by operating on the state object or indirectly by selecting it with selectors.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// directly from state object
state.something;

// indirectly from state object via selector
const getSomething = (state) => state.something;
~~~~~~~~

State can be altered by dispatching an action directly or by using an action creator that returns an action object.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// dispatching an action directly
dispatch({ type: 'ANY_TYPE', payload: anyPayload });

// dispatching an action indirectly via action creator
function doAnything(payload) {
  return {
    type: 'ANY_TYPE',
    payload,
  };
}

dispatch(doAnything(anyPayload));
~~~~~~~~

In order to keep your state predictable and manageable in the reducers, you can apply techniques for an improved state structure. You can normalize your state to have always a single source of truth. That means you don't have to operate on duplicated entities, but only on one reference of the entity. In addition, it keeps the state flat. It is easier to manage only by using JavaScript spread operators for keeping it immutable.

Around these practical usages, you have learned several supporting techniques. There are tons of opinionated ways to organize your folders and files. The book showcased two of the main approaches, but they vary in their execution from developer to developer, team to team, or company to company. Nevertheless, you should always bear in mind to keep Redux at a top level. It is not used to manage the state of one single component. Instead it is used to wire dedicated components to the store in order to enable them to alter and to retrieve the state from it.

Coupling actions and reducers is fine, but always think twice when adding another action type. For instance, perhaps a action type could be reused in another reducer. When reusing action types, you avoid to end up with fat thunks when using Redux thunk. Instead of dispatching several actions, your thunk could dispatch only one abstract action that is reused in more than one reducer.

You have learned that you can plan your state management ahead. There are use cases where local state makes more sense than sophisticated state. Both can be used and should be used in a scaling application. By combining local state to the native local storage of the browser, you can give the user of your application an improved UX. In addition, you can plan the state ahead too. Think about view state and entity state and where it should live in your application. You can give your reducers difference domains as their ownership such as `todoReducer`, `filterReducer` and `notificationReducer`. However, once you have planned your state management and state, don't stick to it. When growing your application, always revisit those things to apply refactorings. That will help you to keep your state manageable, maintainable and predictable in the long run.

## Hands On: Hacker News with Redux

In this chapter, you will be guided to build your own [Hacker News](https://news.ycombinator.com/) application with React and Redux. Hacker News is a platform to share news around the technology domain. It provides a [public API](https://hn.algolia.com/api) to retrieve their data. Some of you might have read [the Road to learn React](https://www.robinwieruch.de/the-road-to-learn-react/) where you have build a Hacker News application as well. In that book it was only plain React. Now you can experience the differences when using Redux with React in this book.

You are going to use create-react-app to setup your project. You can read the [official documentation](https://github.com/facebookincubator/create-react-app) to get to know how it works. After you have installed it, you simply start by choosing a project name for your application.

{title="Command Line",lang="text"}
~~~~~~~~
create-react-app react-redux-hackernews
~~~~~~~~

After the project was created for you, you can navigate into the project folder, open your editor and start the application.

{title="Command Line",lang="text"}
~~~~~~~~
cd react-redux-hackernews
npm start
~~~~~~~~

In your browser it should show the defaults that come with create-create-app.

### Part 1: Project Organization

Before you familiarize yourself with the folder structure in this part, you will adapt it to your own needs. First, navigate into the *src/* folder and delete the boilerplate files that are not needed for the application.

{title="Command Line: /",lang="text"}
~~~~~~~~
cd src
rm logo.svg App.js App.test.js App.css
~~~~~~~~

Even the `App` component is removed, because you'll organize it in folders instead of in the top level *src/* folder. Now, from the *src/* folder, create the folders for an organized folder structure by a technical separation. It is up to you to refactor it later to a feature folder organization.

{title="Command Line: src/",lang="text"}
~~~~~~~~
mkdir components reducers actions selectors store sagas api constants
~~~~~~~~

Your folder structure should be similar to the following:

{title="Folder Structure",lang="text"}
~~~~~~~~
-src/
--actions/
--api/
--components/
--constants/
--reducers/
--sagas/
--selectors/
--store/
--index.css
--index.js
~~~~~~~~

Navigate in the *components/* folder and create the following files for your independent components. These are not all components yet. You will create more of them on your own for this application afterward.

{title="Command Line: src/",lang="text"}
~~~~~~~~
cd components
touch App.js Stories.js Story.js App.css Stories.css Story.css
~~~~~~~~

You can continue this way and create the remaining files to end up with the following folder structure.

{title="Folder Structure",lang="text"}
~~~~~~~~
-src/
--actions/
--api/
--components/
---App.js
---App.css
---Stories.js
---Stories.css
---Story.js
---Story.css
--constants/
---actionTypes.js
--reducers/
---index.js
--sagas/
---index.js
--selectors/
--store/
---index.js
--index.css
--index.js
~~~~~~~~

Now you have your foundation of folders and files for your React and Redux application. Except for the specific component files that you already have, everything else can be used as a blueprint, your own boilerplate, for any application using React and Redux. But only if it is separated by technical concerns. In a growing application, you might want to separate your folders by feature. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/d5ab6a77653ee641d339c0a6a91c8444eff3f699).

### Part 2: Plain React Components

In this part you will implement your plain React component architecture that only receives all necessary props from their parent components. These props can include callback functions that will enable interactions later on. The point is that the props don't reveal where they are coming from. They could be props themselves that are located in the parent component, state from the local state or even Redux state. The callback functions are plain functions too. Thus the components are not aware of using local state methods or Redux actions to alter the state.

In your entry point to React, where your root component gets rendered into the DOM, adjust the import of the `App` component by including the components folder in the path.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import App from './components/App';
# leanpub-end-insert
import './index.css';

ReactDOM.render(<App />, document.getElementById('root'));
~~~~~~~~

In the next step, you can come up with sample data that can be used in the React components. The sample data becomes the input of the `App` component. At a later point in time, this data will get fetched from the Hacker News API.

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const stories = [
  {
    title: 'React',
    url: 'https://facebook.github.io/react/',
    author: 'Jordan Walke',
    num_comments: 3,
    points: 4,
    objectID: 0,
  }, {
    title: 'Redux',
    url: 'https://github.com/reactjs/redux',
    author: 'Dan Abramov, Andrew Clark',
    num_comments: 2,
    points: 5,
    objectID: 1,
  },
];

ReactDOM.render(
  <App stories={stories} />,
  document.getElementById('root')
);
# leanpub-end-insert
~~~~~~~~

The three components, `App`, `Stories` and `Story`, are not defined yet but you have already created the files. Let's define them component by component. First, the `App` component receives the sample stories from above as props and its only responsibility is to render the `Stories` component and to pass over the `stories` as props.

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './App.css';

import Stories from './Stories';

const App = ({ stories }) =>
  <div className="app">
    <Stories stories={stories} />
  </div>

export default App;
~~~~~~~~

Second, the `Stories` component receives the `stories` as props and renders for each story a `Story` component. You may want to default to an empty array that the `Stories` component doesn't crash when the list of stories is null.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './Stories.css';

import Story from './Story';

const Stories = ({ stories }) =>
  <div className="stories">
    {(stories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
      />
    )}
  </div>

export default Stories;
~~~~~~~~

Third, the `Story` component renders a few properties of the `story` object. The story object gets already destructured from the props in the function signature. Furthermore the story object gets destructured as well.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './Story.css';

const Story = ({ story }) => {
  const {
    title,
    url,
    author,
    num_comments,
    points,
  } = story;

  return (
    <div className="story">
      <span>
        <a href={url}>{title}</a>
      </span>
      <span>{author}</span>
      <span>{num_comments}</span>
      <span>{points}</span>
    </div>
  );
}

export default Story;
~~~~~~~~

You can start your application again with `npm start` on the command line. Both sample stories should be displayed in plain React. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/f5843d2a06033cd045e6d0427993e30e289031a7).

### Part 3: Apply Styling

The application looks a bit dull without any styling. Therefore you can drop in some of your own styling or use the styling that's provided in this part. First, the application would need some general style that can be defined in the root style file.

{title="src/index.css",lang="css"}
~~~~~~~~
body {
  color: #222;
  background: #f4f4f4;
  font: 400 14px CoreSans, Arial,sans-serif;
}

a {
  color: #222;
}

a:hover {
  text-decoration: underline;
}

ul, li {
  list-style: none;
  padding: 0;
  margin: 0;
}

input {
  padding: 10px;
  border-radius: 5px;
  outline: none;
  margin-right: 10px;
  border: 1px solid #dddddd;
}

button {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #dddddd;
  background: transparent;
  color: #808080;
  cursor: pointer;
}

button:hover {
  color: #222;
}

.button-inline {
  border-width: 0;
  background: transparent;
  color: inherit;
  text-align: inherit;
  -webkit-font-smoothing: inherit;
  padding: 0;
  font-size: inherit;
  cursor: pointer;
}

.button-active {
  border-radius: 0;
  border-bottom: 1px solid #38BB6C;
}

*:focus {
  outline: none;
}
~~~~~~~~

Second, the `App` component gets a few CSS classes:

{title="src/components/App.css",lang="css"}
~~~~~~~~
.app {
  margin: 20px;
}

.interactions, .error {
  text-align: center;
}
~~~~~~~~

Third, the `Stories` component gets some style:

{title="src/components/Stories.css",lang="css"}
~~~~~~~~
.stories {
  margin: 20px 0;
}

.stories-header {
  display: flex;
  line-height: 24px;
  font-size: 16px;
  padding: 0 10px;
  justify-content: space-between;
}

.stories-header > span {
  overflow: hidden;
  text-overflow: ellipsis;
  padding: 0 5px;
}
~~~~~~~~

And last but not least, the `Story` component will get styled too:

{title="src/components/Story.css",lang="css"}
~~~~~~~~
.story {
  display: flex;
  line-height: 24px;
  white-space: nowrap;
  margin: 10px 0;
  padding: 10px;
  background: #ffffff;
  border: 1px solid #e3e3e3;
}

.story > span {
  overflow: hidden;
  text-overflow: ellipsis;
  padding: 0 5px;
}
~~~~~~~~

When you start your application again, it seems more organized by its styling. But there is still something missing for displaying the stories properly. The columns for each story should be aligned and perhaps there should be a heading for each column. First, you can define an object to describe the columns.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './Stories.css';

import Story from './Story';

# leanpub-start-insert
const COLUMNS = {
  title: {
    label: 'Title',
    width: '40%',
  },
  author: {
    label: 'Author',
    width: '30%',
  },
  comments: {
    label: 'Comments',
    width: '10%',
  },
  points: {
    label: 'Points',
    width: '10%',
  },
  archive: {
    width: '10%',
  },
};
# leanpub-end-insert

const Stories = ({ stories }) =>
  ...
~~~~~~~~

The last column with the `archive` property name will not be used yet, but will be used in a later point in time. Second, you can pass this object to your `Story` component. Still the `Stories` component has access to the object to use it later on for the column headings.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
const Stories = ({ stories }) =>
  <div className="stories">
    {(stories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
# leanpub-start-insert
        columns={COLUMNS}
# leanpub-end-insert
      />
    )}
  </div>
~~~~~~~~

The `Story` component can use the columns object to style each displaying property of the story. It uses inline style to define the width of each column which comes from the object.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const Story = ({ story, columns }) => {
# leanpub-end-insert

  ...

  return (
    <div className="story">
# leanpub-start-insert
      <span style={{ width: columns.title.width }}>
        <a href={url}>{title}</a>
      </span>
      <span style={{ width: columns.author.width }}>
        {author}
      </span>
      <span style={{ width: columns.comments.width }}>
        {num_comments}
      </span>
      <span style={{ width: columns.points.width }}>
        {points}
      </span>
      <span style={{ width: columns.archive.width }}>
      </span>
# leanpub-end-insert
    </div>
  );
}
~~~~~~~~

Last but not least, you can use the `COLUMNS` object to give your `Stories` component matching header columns. That's why the `COLUMNS` object got defined in the `Stories` component in the first place. Now, rather than doing it manually, as in the `Story` component, you will map over the object dynamically to render the header columns. Since it is an object, you have to turn it into an array of the property names first, and then access the object by its mapped keys again.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
const Stories = ({ stories }) =>
  <div className="stories">
# leanpub-start-insert
    <div className="stories-header">
      {Object.keys(COLUMNS).map(key =>
        <span
          key={key}
          style={{ width: COLUMNS[key].width }}
        >
          {COLUMNS[key].label}
        </span>
      )}
    </div>
# leanpub-end-insert

    {(stories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
        columns={COLUMNS}
      />
    )}
  </div>
~~~~~~~~

You can extract the header columns as its own `StoriesHeader` component to keep your components well arranged and separated by concerns.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
const Stories = ({ stories }) =>
  <div className="stories">
# leanpub-start-insert
    <StoriesHeader columns={COLUMNS} />
# leanpub-end-insert

    {(stories || []).map(story =>
      ...
    )}
  </div>

# leanpub-start-insert
const StoriesHeader = ({ columns }) =>
  <div className="stories-header">
    {Object.keys(columns).map(key =>
      <span
        key={key}
        style={{ width: columns[key].width }}
      >
        {columns[key].label}
      </span>
    )}
  </div>
# leanpub-end-insert
~~~~~~~~

In this part, you have applied styling for your application and components. It should be in a representable state from a developer's point of view. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/6cb35b024abb59a2192c8ac0bb700046a700d470).

### Part 4: Archive a Story

Now you will add your first functionality: archiving a story. Therefore you will have to introduce Redux at some point to your application to manage the state of archived stories. I want to highly emphasize that it would work in plain React too. But for the sake of learning Redux, you will already use it at this point in time.

First, the archiving functionality can be passed down to the `Story` component from your React root component. In the beginning, it can be an empty function. The function will be replaced later when you will dispatch a Redux action.

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

ReactDOM.render(
# leanpub-start-insert
  <App stories={stories} onArchive={() => {}} />,
# leanpub-end-insert
  document.getElementById('root')
);
~~~~~~~~

Second, you can pass it through your `App` and `Stories` components. These components don't use the function but only pass it to the `Story` component. You might already notice that this could be a potential refactoring later on, because the function gets passed from the root component through a few components only to reach a leaf component. It passes the `App` component:

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const App = ({ stories, onArchive }) =>
# leanpub-end-insert
  <div className="app">
    <Stories
      stories={stories}
# leanpub-start-insert
      onArchive={onArchive}
# leanpub-end-insert
    />
  </div>
~~~~~~~~

And it passes the `Stories` component:

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const Stories = ({ stories, onArchive }) =>
# leanpub-end-insert
  <div className="stories">
    <StoriesHeader columns={COLUMNS} />

    {(stories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
        columns={COLUMNS}
# leanpub-start-insert
        onArchive={onArchive}
# leanpub-end-insert
      />
    )}
  </div>
~~~~~~~~

Finally, you can use it in your `Story` component in a `onClick` handler of a button. The story `objectID` will be passed in the handler to identify the archived story.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const Story = ({ story, columns, onArchive }) => {
# leanpub-end-insert
  const {
    title,
    url,
    author,
    num_comments,
    points,
# leanpub-start-insert
    objectID,
# leanpub-end-insert
  } = story;

  return (
    <div className="story">
      ...
      <span style={{ width: columns.archive.width }}>
# leanpub-start-insert
        <button
          type="button"
          className="button-inline"
          onClick={() => onArchive(objectID)}
        >
          Archive
        </button>
# leanpub-end-insert
      </span>
    </div>
  );
}
~~~~~~~~

A refactoring that you could already do would be to extract the button as a reusable component.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
const Story = ({ story, columns, onArchive }) => {
  ...

  return (
    <div className="story">
      ...
      <span style={{ width: columns.archive.width }}>
# leanpub-start-insert
        <ButtonInline onClick={() => onArchive(objectID)}>
          Archive
        </ButtonInline>
# leanpub-end-insert
      </span>
    </div>
  );
}

# leanpub-start-insert
const ButtonInline = ({
  onClick,
  type = 'button',
  children
}) =>
  <button
    type={type}
    className="button-inline"
    onClick={onClick}
  >
    {children}
  </button>
# leanpub-end-insert
~~~~~~~~

You can make even another more abstract `Button` component that doesn't share the `button-inline` CSS class.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
...

const ButtonInline = ({
  onClick,
  type = 'button',
  children
}) =>
  <Button
    type={type}
    className="button-inline"
    onClick={onClick}
  >
    {children}
  </Button>

const Button = ({
  onClick,
  className,
  type = 'button',
  children
}) =>
  <button
    type={type}
    className={className}
    onClick={onClick}
  >
    {children}
  </button>
~~~~~~~~

Both button components should be extracted to a new file called *src/components/Button.js*, but exported so that at least the `ButtonInline` component can be reused in the `Story` component. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/55de13475aa9c2424b0fc00ce95dd4c5474c0473). Now, when you start your application again, the button to archive a story is there. But it doesn't work because it only receives a no-op (empty function) as property from your React root component. Later you will introduce a Redux action that can be dispatched from this function to archive a story.

### Part 5: Introduce Redux: Store + First Reducer

This part will finally introduce Redux to manage the state of the (sample) stories instead of passing it directly into your component tree. Let's approach it step by step. First, you have to install Redux on the command line:

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux
~~~~~~~~

Second, in the root entry point of React, you can import the Redux store. The store is not defined yet. Instead of using the sample stories, you will use the stories that are stored in the Redux store. Taken that the store only saves a list of stories as state, you can simply get the root state of the store and assume that it is the list of stories.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
# leanpub-start-insert
import store from './store';
# leanpub-end-insert
import './index.css';

ReactDOM.render(
# leanpub-start-insert
  <App stories={store.getState()} onArchive={() => {}} />,
# leanpub-end-insert
  document.getElementById('root')
);
~~~~~~~~

Third, you have to create your Redux store instance in a separate file. It already takes a reducer that is not implemented yet. You will implement it in the next step.

{title="src/store/index.js",lang="javascript"}
~~~~~~~~
import { createStore } from 'redux';
import storyReducer from '../reducers/story';

const store = createStore(
  storyReducer
);

export default store;
~~~~~~~~

Fourth, in your *src/reducers/* folder you can create your first reducer: `storyReducer`. It can have the sample stories as initial state.

{title="src/reducers/story.js",lang="javascript"}
~~~~~~~~
const INITIAL_STATE = [
  {
    title: 'React',
    url: 'https://facebook.github.io/react/',
    author: 'Jordan Walke',
    num_comments: 3,
    points: 4,
    objectID: 0,
  }, {
    title: 'Redux',
    url: 'https://github.com/reactjs/redux',
    author: 'Dan Abramov, Andrew Clark',
    num_comments: 2,
    points: 5,
    objectID: 1,
  },
];

function storyReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
    default : return state;
  }
}

export default storyReducer;
~~~~~~~~

Your application should work when you start it. It is using the state from the Redux store that is initialized in the `storyReducer`, because it is the only reducer in your application. There are no actions yet and no action is captured in the reducer yet. Even though there was no action dispatched, you can see that the Redux store runs through all its defined reducers to initialize its initial state in the store. The state gets visible through the `Stories` and `Story` components, because it is passed down from the React root entry point. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/5aafb21595541c21db778ad8825c97403e44b963).

### Part 6: Two Reducers

You have used the Redux store and a reducer to define an initial state of sample stories and to retrieve this state for your component tree. But there is no state manipulation happening yet. In the following parts you are going to implement the archive functionality. When approaching this functionality, the simplest thing to do would be to remove the archived story from the list of stories in the `storyReducer`. But let's approach this from a different angle to have a greater impact in the long run. It could still be useful to have all stories in the end, but have a way to distinguish between them: stories and archived stories. Following this way, you would be able in the future to have a second component that shows the archived stories next to the available stories.

From an implementation point of view, the `storyReducer` will stay as it is for now. But you can introduce a second reducer, a `archiveReducer`, that keeps a list of references to the archived stories.

{title="src/reducers/archive.js",lang="javascript"}
~~~~~~~~
const INITIAL_STATE = [];

function archiveReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
    default : return state;
  }
}

export default archiveReducer;
~~~~~~~~

You will implement the action to archive a story in a second. First, the Redux store in its instantiation needs to get both reducers now. It has to get the combined reducer. Let's pretend that the store can import the combined reducer from the entry file, the *reducers/index.js*, without worrying about the combining of the reducers yet.

{title="src/store/index.js",lang="javascript"}
~~~~~~~~
import { createStore } from 'redux';
# leanpub-start-insert
import rootReducer from '../reducers';
# leanpub-end-insert

const store = createStore(
# leanpub-start-insert
  rootReducer
# leanpub-end-insert
);

export default store;
~~~~~~~~

Next you can combine both reducers in the file that is used by the Redux store to import the `rootReducer`.

{title="src/reducers/index.js",lang="javascript"}
~~~~~~~~
import { combineReducers } from 'redux';
import storyReducer from './story';
import archiveReducer from './archive';

const rootReducer = combineReducers({
  storyState: storyReducer,
  archiveState: archiveReducer,
});

export default rootReducer;
~~~~~~~~

Since your state is sliced up into two substates now, you have to adjust how you retrieve the stories from your store with the intermediate `storyState`. This is a crucial step, because it shows how a combined reducer slices up your state into substates.

{title="src/index.js",lang="javascript"}
~~~~~~~~
ReactDOM.render(
  <App
# leanpub-start-insert
    stories={store.getState().storyState}
# leanpub-end-insert
    onArchive={() => {}}
  />,
  document.getElementById('root')
);
~~~~~~~~

The application should show up the same stories as before when you start it. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/f6d436fdfdab19296e473fbe7243690e830c1c2b). However, there is still no state manipulation happening, because no actions are involved yet. Finally in the next part you will dispatch your first action to archive a story.

### Part 7: First Action

In this part, you will dispatch your first action to archive a story. The archive action needs to be captured in the new `archiveReducer`. It simply stores all archived stories by their id in a list. There is no need to duplicate the story entity, because you want to keep the law of a single source of truth. The initial state is an empty list, because no story is archived in the beginning. When archiving a story, all the previous ids in the state and the new archived id will be used in a new array. The JavaScript spread operator is used here.

{title="src/reducers/archive.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { STORY_ARCHIVE } from '../constants/actionTypes';
# leanpub-end-insert

const INITIAL_STATE = [];

# leanpub-start-insert
const applyArchiveStory = (state, action) =>
  [ ...state, action.id ];
# leanpub-end-insert

function archiveReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
# leanpub-start-insert
    case STORY_ARCHIVE : {
      return applyArchiveStory(state, action);
    }
# leanpub-end-insert
    default : return state;
  }
}

export default archiveReducer;
~~~~~~~~

The action type is already outsourced in a different file. This way it can be reused when dispatching the action from the Redux store.

{title="src/constants/actionTypes.js",lang="javascript"}
~~~~~~~~
export const STORY_ARCHIVE = 'STORY_ARCHIVE';
~~~~~~~~

Last but not least, you can import the action type and dispatch the action in your root component where you had the empty function before.

{title="src/reducers/archive.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
import store from './store';
# leanpub-start-insert
import { STORY_ARCHIVE } from './constants/actionTypes';
# leanpub-end-insert
import './index.css';

ReactDOM.render(
  <App
    stories={store.getState().storyState}
# leanpub-start-insert
    onArchive={id => store.dispatch({ type: STORY_ARCHIVE, id })}
# leanpub-end-insert
  />,
  document.getElementById('root')
);
~~~~~~~~

Now you dispatch the action directly without an action creator. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/5ddbcc2fa8269d615763770a49e7675c5f02d173). When you start your application, it should still work, but nothing happens when you archive a story. The archived stories are not yet evaluated in the component tree. The `stories` prop that is passed to the `App` component still uses all the stories from the `storyState`.

### Part 8: First Selector

You can use both substates, `storyState` and `archiveState` to derive the list of stories that are not archived. The deriving of those properties can happen in a selector. You can create your first selector that only returns the part of the stories that is not archived. The `archiveState` is the list of archived ids.

{title="src/selectors/story.js",lang="javascript"}
~~~~~~~~
const isNotArchived = archivedIds => story =>
  archivedIds.indexOf(story.objectID) === -1;

const getReadableStories = ({ storyState, archiveState }) =>
  storyState.filter(isNotArchived(archiveState));

export {
  getReadableStories,
};
~~~~~~~~

The selector makes heavily use of JavaScript ES6 arrow functions, JavaScript ES6 destructuring and a higher-order function: `isNotArchived()`. If you are not used to JavaScript ES6, don't feel intimidated by it. It is only a way to express these functions more concise. In plain JavaScript ES5 it would look like the following:

{title="src/selectors/story.js",lang="javascript"}
~~~~~~~~
function isNotArchived(archivedIds) {
  return function (story) {
    return archivedIds.indexOf(story.objectID) === -1;
  };
}

function getReadableStories({ storyState, archiveState }) {
  return storyState.filter(isNotArchived(archiveState));
}

export {
  getReadableStories,
};
~~~~~~~~

Last but not least, you can use the selector to compute the not archived stories instead of retrieving the whole list of stories from the store directly.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
import store from './store';
# leanpub-start-insert
import { getReadableStories } from './selectors/story';
# leanpub-end-insert
import { STORY_ARCHIVE } from './constants/actionTypes';
import './index.css';

ReactDOM.render(
  <App
# leanpub-start-insert
    stories={getReadableStories(store.getState())}
# leanpub-end-insert
    onArchive={id => store.dispatch({ type: STORY_ARCHIVE, id })}
  />,
  document.getElementById('root')
);
~~~~~~~~

You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/5e3338d3ffff924b7a12eccb691365fd11cb5aed). When you start your application, nothing happens again when you archive a story. Even though you are using the readable stories now. That's because there is no re-rendering of the view in place to update it.

### Part 9: Re-render View

In this part, you will update the view layer to reflect the correct state that is used from the Redux store. When an action dispatches, the state in the Redux store gets updated. However, the component tree in React doesn't update, because no one subscribed to the Redux store yet. In the first attempt, you are going to wire up Redux and React naively and re-render the whole component tree on each updatea as you have done before in another application from this book.

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
function render() {
# leanpub-end-insert
  ReactDOM.render(
    <App
      stories={getReadableStories(store.getState())}
      onArchive={id => store.dispatch({ type: STORY_ARCHIVE, id })}
    />,
    document.getElementById('root')
  );
# leanpub-start-insert
}
# leanpub-end-insert

# leanpub-start-insert
store.subscribe(render);
render();
# leanpub-end-insert
~~~~~~~~

Now the components will re-render once you archive a story, because the state in the Redux store updates and the subscription will run to render again the whole component tree. In addition, you render the component only once when the application starts. Congratulations, you dispatched your first action, selected derived properties from the state and updated your component tree by subscribing it to the Redux store. That took longer as expected, didn't it? However, now most of the Redux and React infrastructure is in place to be more efficient when introducing new features. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/286c04354fcab639ebd60ac2430ad939ce107365).

### Part 10: First Middleware

In this part, you will introduce your first middleware to the Redux store. In a scaling application it becomes often a problem to track state updates. Often you don't notice when an action is dispatched, because too many actions get involved and a bunch of them might get triggered implicitly. Therefore you can use the [redux-logger](https://github.com/evgenyrodionov/redux-logger) middleware in your Redux store to `console.log()` every action, the previous state and the next state, automatically to your developers console when dispatching an action. First, you have to install the neat middleware library.

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux-logger
~~~~~~~~

Second, you can use it as middleware in your Redux store initialization.

{title="src/store/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { createStore, applyMiddleware } from 'redux';
import { createLogger } from 'redux-logger';
# leanpub-end-insert
import rootReducer from '../reducers';

# leanpub-start-insert
const logger = createLogger();
# leanpub-end-insert

const store = createStore(
  rootReducer,
# leanpub-start-insert
  undefined,
  applyMiddleware(logger)
# leanpub-end-insert
);

export default store;
~~~~~~~~

That's it. Every time you dispatch an action now, for instance when archiving a story, you will see the logging in the developer console in your browser. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/652e6419e2a872ba2d1dd65465006b13f0799c4f).

### Part 11: First Action Creator

The action you are dispatching is a plain action object. However, you might want to reuse it in a later point in time. Action creators are not mandatory, but they keep your Redux architecture organized. In order to stay organized, let's define your first action creator. First, you have to define the action creator that takes a story id, to identify the archiving story, in a new file.

{title="src/actions/archive.js",lang="javascript"}
~~~~~~~~
import { STORY_ARCHIVE } from '../constants/actionTypes';

const doArchiveStory = id => ({
  type: STORY_ARCHIVE,
  id,
});

export {
  doArchiveStory,
};
~~~~~~~~

Second, you can use it in your root component. Instead of dispatching the action object directly, you can create an action by using its action creator.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
import store from './store';
import { getReadableStories } from './selectors/story';
# leanpub-start-insert
import { doArchiveStory } from './actions/archive';
# leanpub-end-insert
import './index.css';

function render() {
  ReactDOM.render(
    <App
      stories={getReadableStories(store.getState())}
# leanpub-start-insert
      onArchive={id => store.dispatch(doArchiveStory(id))}
# leanpub-end-insert
    />,
    document.getElementById('root')
  );
}

...
~~~~~~~~

The application should operate as before when you start it. But this time you have used an action creator rather than dispatching an action object directly. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/4cc5e995d63fd935a2e335b0a4946a1811c04202).

### Part 12: Connect React with Redux

In this part, you will connect React and Redux in a more sophisticated way. The whole component tree re-renders every time when the state changes now. However, you might want to wire up components independently with the Redux store. In addition, you don't want to re-render the whole component tree, but only the components where the state or props have changed. Let's change this by using the [react-redux](https://github.com/reactjs/react-redux) library that connects both worlds.

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save react-redux
~~~~~~~~

You can use the `Provider` component, which makes the Redux store available to all components below, in your React root entry point.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import { Provider } from 'react-redux';
# leanpub-end-insert
import App from './components/App';
import store from './store';
import './index.css';

# leanpub-start-insert
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
# leanpub-end-insert
~~~~~~~~

Notice that the render method isn't used in a Redux store subscription anymore. No one subscribes to the Redux store and the `App` component isn't receiving any props. In addition, the `App` component is only rendering a component and doesn't pass any props.

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './App.css';

import Stories from './Stories';

# leanpub-start-insert
const App = () =>
# leanpub-end-insert
  <div className="app">
# leanpub-start-insert
    <Stories />
# leanpub-end-insert
  </div>

export default App;
~~~~~~~~

But who gives the props to the `Stories` component then? This component is the first component that needs to know about the list of stories, because it has to display it. The solution is to upgrade the `Stories` component to a connected component. It should be connected to the state layer. So, instead of only exporting the plain `Stories` component:

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
...

export default Stories;
~~~~~~~~

You can export the connected component that has access to the Redux store:

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { connect } from 'react-redux';
import { doArchiveStory } from '../actions/archive';
import { getReadableStories } from '../selectors/story';
# leanpub-end-insert

...

# leanpub-start-insert
const mapStateToProps = state => ({
  stories: getReadableStories(state),
});

const mapDispatchToProps = dispatch => ({
  onArchive: id => dispatch(doArchiveStory(id)),
});

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Stories);
# leanpub-end-insert
~~~~~~~~

The `Stories` component is a connected component now and is the only component that has access to the Redux store. It receives the stories from the state in `mapStateToProps()` and a function that triggers the dispatching of an action to archive a story in `mapDispatchToProps()`. The application should work again, but this time with a sophisticated connection between Redux and React. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/88072e9b62230f59ffa83a5ddd06ceda6bf75fe4).

### Part 13: Lift Connection

It is no official term (yet), but you can lift the connection between React and Redux. For instance, you could lift the connection from the `Stories` component to another component. But you need the list of stories to map over them in the `Stories` component. However, what about the `onArchive()` function? It is not used in the `Stories` component, but only in the `Story` component and only passed via the `Stories` component. Thus you could lift the connection partly. The `stories` would stay in the `Stories` component, but the `onArchive()` function could live in the `Story` component.

First, you remove the `onArchive()` function for the `Stories` component and remove the `mapDispatchToProps()` as well. It will be used later on in the `Story` component.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const Stories = ({ stories }) =>
# leanpub-end-insert
  <div className="stories">
    <StoriesHeader columns={COLUMNS} />

    {(stories || []).map(story =>
# leanpub-start-insert
      <Story
        key={story.objectID}
        story={story}
        columns={COLUMNS}
      />
# leanpub-end-insert
    )}
  </div>

...

const mapStateToProps = state => ({
  stories: getReadableStories(state),
});

# leanpub-start-insert
export default connect(
  mapStateToProps
)(Stories);
# leanpub-end-insert
~~~~~~~~

Now you can connect the `Story` component instead whereas you have two connected components afterward.

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { connect } from 'react-redux';
import { doArchiveStory } from '../actions/archive';
# leanpub-end-insert

...

# leanpub-start-insert
const mapDispatchToProps = dispatch => ({
  onArchive: id => dispatch(doArchiveStory(id)),
});

export default connect(
  null,
  mapDispatchToProps
)(Story);
# leanpub-end-insert
~~~~~~~~

With this refactoring step in your mind, you can always lift your connections to the Redux store in your view layer depending on the needs of the components. Does the component need state from the Redux store? Does the component need to alter the state in the Redux store via dispatching an action? You are in full control of where you want to use connected components (a subset of container components) and where you want to keep your components as presenter components. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/779d52fc85ecfbaf5a821cbbae384aac962e76a7).

### Part 14: Interacting with an API

Implementing applications with sample data can be dull. It is way more exciting to interact with a real API - in this case the [Hacker News API](https://hn.algolia.com/api). Even though, as you have learned, you can have asynchronous actions without any asynchronous action library, this application will introduce Redux Saga as asynchronous action library to deal with side-effects such as fetching data from a third-party platform.

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save redux-saga
~~~~~~~~

First, you can introduce a root saga in your entry point file to sagas. It can be similar seen to the combined root reducer, because in the end the Redux store expects one root saga for its creation. Basically the root saga watches all saga activated actions by using effects such as `takeEvery()`.

{title="src/sagas/index.js",lang="javascript"}
~~~~~~~~
import { takeEvery, all } from 'redux-saga/effects';
import { STORIES_FETCH } from '../constants/actionTypes';
import { handleFetchStories } from './story';

function *watchAll() {
  yield all([
    takeEvery(STORIES_FETCH, handleFetchStories),
  ])
}

export default watchAll;
~~~~~~~~

Second, the root saga can be used in the Redux store middleware when initializing the saga middleware. It is used in the middleware, but also needs to be run in a `saga.run()` method.

{title="src/store/index.js",lang="javascript"}
~~~~~~~~
import { createStore, applyMiddleware } from 'redux';
import { createLogger } from 'redux-logger';
# leanpub-start-insert
import createSagaMiddleware from 'redux-saga';
# leanpub-end-insert
import rootReducer from '../reducers';
# leanpub-start-insert
import rootSaga from '../sagas';
# leanpub-end-insert

const logger = createLogger();
# leanpub-start-insert
const saga = createSagaMiddleware();
# leanpub-end-insert

const store = createStore(
  rootReducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(saga, logger)
# leanpub-end-insert
);

# leanpub-start-insert
saga.run(rootSaga);
# leanpub-end-insert

export default store;
~~~~~~~~

Third, you can introduce the new action type in your constants that will activate the saga. However, you can already introduce a second action type that will later on - when the request succeeds - add the stories in your `storyReducer` to the Redux store. Basically you have one action to activate the side-effect that is handled with Redux Saga and one action that stores the result of the side-effect in the Redux store.

{title="src/constants/actionTypes.js",lang="javascript"}
~~~~~~~~
export const STORY_ARCHIVE = 'STORY_ARCHIVE';
# leanpub-start-insert
export const STORIES_FETCH = 'STORIES_FETCH';
export const STORIES_ADD = 'STORIES_ADD';
# leanpub-end-insert
~~~~~~~~

Fourth, you can implement the story saga that encapsulates the API request. It uses the native fetch API of the browser to retrieve the stories from the Hacker News API endpoint. In your `handleFetchStories()` generator function, that is used in your root saga, you can use the `yield` statement to write asynchronous code as it would be synchronous code. As long as the promise from the Hacker News request doesn't resolve (or reject), the next line of code after the `yield` state will not be evaluated. When you finally have the result from the API request, you can use the `put()` effect to dispatch another action.

{title="src/sagas/story.js",lang="javascript"}
~~~~~~~~
import { call, put } from 'redux-saga/effects';
import { doAddStories } from '../actions/story';

const HN_BASE_URL = 'http://hn.algolia.com/api/v1/search?query=';

const fetchStories = query =>
  fetch(HN_BASE_URL + query)
    .then(response => response.json());

function* handleFetchStories(action) {
  const { query } = action;
  const result = yield call(fetchStories, query);
  yield put(doAddStories(result.hits));
}

export {
  handleFetchStories,
};
~~~~~~~~

In the fifth step, you need to define both actions creators: the first one that activates the side-effect to fetch stories by a search term and the second one that adds the fetched stories to your Redux store.

{title="src/actions/story.js",lang="javascript"}
~~~~~~~~
import {
  STORIES_ADD,
  STORIES_FETCH,
} from '../constants/actionTypes';

const doAddStories = stories => ({
  type: STORIES_ADD,
  stories,
});

const doFetchStories = query => ({
  type: STORIES_FETCH,
  query,
});

export {
  doAddStories,
  doFetchStories,
};
~~~~~~~~

Only the second action needs to be intercepted in your `storyReducer` to store the stories. The first action is only used to activate the saga in your root saga. Don't forget to remove the sample stories in your reducers, because they are coming from the API now.

{title="src/reducers/story.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { STORIES_ADD } from '../constants/actionTypes';

const INITIAL_STATE = [];

const applyAddStories = (state, action) =>
  action.stories;
# leanpub-end-insert

function storyReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
# leanpub-start-insert
    case STORIES_ADD : {
      return applyAddStories(state, action);
    }
# leanpub-end-insert
    default : return state;
  }
}

export default storyReducer;
~~~~~~~~

Now, everything is setup from a Redux and Redux Saga perspective. As last step, only one component from the view layer needs to activate the `STORIES_FETCH` action. This action is intercepted in the saga, fetches the stories in a side-effect, and stores them in the Redux store with the other `STORIES_ADD` action. Therefore, in your `App` component, you can introduce the new `SearchStories` component.

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import './App.css';

import Stories from './Stories';
# leanpub-start-insert
import SearchStories from './SearchStories';
# leanpub-end-insert

const App = () =>
  <div className="app">
# leanpub-start-insert
    <div className="interactions">
      <SearchStories />
    </div>
# leanpub-end-insert
    <Stories />
  </div>

export default App;
~~~~~~~~

The `SearchStories` component will be a connected component. It is the next step to implement this component. First, you start with a plain React component that has a form, input field and button.

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import Button from './Button';

class SearchStories extends Component {
  constructor(props) {
    super(props);

    this.state = {
      query: '',
    };
  }

  render() {
    return (
      <form onSubmit={this.onSubmit}>
        <input
          type="text"
          value={this.state.query}
          onChange={this.onChange}
        />
        <Button type="submit">
          Search
        </Button>
      </form>
    );
  }
}

export default SearchStories;
~~~~~~~~

There are two missing class methods: `onChange()` and `onSubmit()`. Let's introduce them to make the component complete.

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
...

# leanpub-start-insert
const applyQueryState = query => () => ({
  query
});
# leanpub-end-insert

class SearchStories extends Component {
  constructor(props) {
    ...

# leanpub-start-insert
    this.onChange = this.onChange.bind(this);
    this.onSubmit = this.onSubmit.bind(this);
# leanpub-end-insert
  }

# leanpub-start-insert
  onSubmit(event) {
    const { query } = this.state;
    if (query) {
      this.props.onFetchStories(query)

      this.setState(applyQueryState(''));
    }

    event.preventDefault();
  }

  onChange(event) {
    const { value } = event.target;
    this.setState(applyQueryState(value));
  }
# leanpub-end-insert

  render() {
    ...
  }
}

export default SearchStories;
~~~~~~~~

The component should work on its own now. It only receives one function from the outside via its props: `onFetchStories()`. This function will dispatch an action to activate the saga that fetches the stories from the Hacker News platform. You would have to connect the `SearchStories` component to make the dispatch functionality available.

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import { connect } from 'react-redux';
import { doFetchStories } from '../actions/story';
# leanpub-end-insert
import Button from './Button';

...

# leanpub-start-insert
const mapDispatchToProps = (dispatch) => ({
  onFetchStories: query => dispatch(doFetchStories(query)),
});

export default connect(
  null,
  mapDispatchToProps
)(SearchStories);
# leanpub-end-insert
~~~~~~~~

Start your application again and try to search for stories such as "React" or "Redux". It should work now. The connect component dispatches an action that activates the saga. The side-effect of the saga is the fetching process of the stories by search term from the Hacker News API. Once the request succeeds, another action gets dispatched and captured in the `storyReducer` to finally store the stories. When using Redux Saga, it is essential to wrap your head around the subject that actions can be used to activate sagas but doesn't need to be evaluated in a reducer. It often happens that another action which is dispatched within the saga is evaluated by the reducers. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/94efe051bd639aeedce402a33af5acb20397f9f2).

### Part 15: Separation of API

There is one refactoring step that you could apply. It would improve the separation between API functionalities and sagas. You would extract the API call from the story saga into an own API folder. Afterward, other sagas can make use of these API requests too. First, extract the functionality from the saga and instead import it.

{title="src/sagas/story.js",lang="javascript"}
~~~~~~~~
import { call, put } from 'redux-saga/effects';
import { doAddStories } from '../actions/story';
# leanpub-start-insert
import { fetchStories } from '../api/story';
# leanpub-end-insert

function* handleFetchStories(action) {
  const { query } = action;
  const result = yield call(fetchStories, query);
  yield put(doAddStories(result.hits));
}

export {
  handleFetchStories,
};
~~~~~~~~

And second, use it in an own dedicated API file.

{title="src/api/story.js",lang="javascript"}
~~~~~~~~
const HN_BASE_URL = 'http://hn.algolia.com/api/v1/search?query=';

const fetchStories = query =>
  fetch(HN_BASE_URL + query)
    .then(response => response.json());

export {
  fetchStories,
};
~~~~~~~~

Great, you have separated the API functionality from the saga. This way you made your API functions reusable to more than one saga. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/b6a6e59af71613471a50c9366c4c4e107e00b66f).

### Part 16: Error Handling

So far, you are making a request to the Hacker News API and display the retrieved stories in your React components. But what happens when an error occurs? Nothing will show up when you search for stories. In order to give your end-user a great user experience, you could add error handling to your application. Let's do it by introducing an action that can allocate an error state in the Redux store.

{title="src/constants/actionTypes.js",lang="javascript"}
~~~~~~~~
export const STORY_ARCHIVE = 'STORY_ARCHIVE';
export const STORIES_FETCH = 'STORIES_FETCH';
# leanpub-start-insert
export const STORIES_FETCH_ERROR = 'STORIES_FETCH_ERROR';
# leanpub-end-insert
export const STORIES_ADD = 'STORIES_ADD';
~~~~~~~~

In the second step, you would need an action creator that keeps an error object in its payload and can be caught in a reducer later on.

{title="src/actions/story.js",lang="javascript"}
~~~~~~~~
import {
  STORIES_ADD,
  STORIES_FETCH,
# leanpub-start-insert
  STORIES_FETCH_ERROR,
# leanpub-end-insert
} from '../constants/actionTypes';

...

# leanpub-start-insert
const doFetchErrorStories = error => ({
  type: STORIES_FETCH_ERROR,
  error,
});
# leanpub-end-insert

export {
  doAddStories,
  doFetchStories,
# leanpub-start-insert
  doFetchErrorStories,
# leanpub-end-insert
};
~~~~~~~~

The action can be called in your story saga now. Redux Saga, because of its generators, uses try and catch statements for error handling. Every time you would get an error in your try block, you would end up in the catch block to do something with the error object. In this case, you can dispatch your new action to store the error state in your Redux store.

{title="src/sagas/story.js",lang="javascript"}
~~~~~~~~
import { call, put } from 'redux-saga/effects';
# leanpub-start-insert
import { doAddStories, doFetchErrorStories } from '../actions/story';
# leanpub-end-insert
import { fetchStories } from '../api/story';

function* handleFetchStories(action) {
  const { query } = action;

# leanpub-start-insert
  try {
# leanpub-end-insert
    const result = yield call(fetchStories, query);
    yield put(doAddStories(result.hits));
# leanpub-start-insert
  } catch (error) {
    yield put(doFetchErrorStories(error));
  }
# leanpub-end-insert
}

export {
  handleFetchStories,
};
~~~~~~~~

Last but not least, a reducer needs to deal with the new action type. The best place to keep it would be next to the stories. The story reducer keeps only a list of stories so far, but you could change it to manage a complex object that holds the list of stories and an error object.

{title="src/reducers/story.js",lang="javascript"}
~~~~~~~~
import { STORIES_ADD } from '../constants/actionTypes';

# leanpub-start-insert
const INITIAL_STATE = {
  stories: [],
  error: null,
};
# leanpub-end-insert

# leanpub-start-insert
const applyAddStories = (state, action) => ({
  stories: action.stories,
  error: null,
});
# leanpub-end-insert

function storyReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
    case STORIES_ADD : {
      return applyAddStories(state, action);
    }
    default : return state;
  }
}

export default storyReducer;
~~~~~~~~

Now you can introduce the second action type in the reducer. It would allocate the error object in the state but keeps the list of stories empty.

{title="src/reducers/story.js",lang="javascript"}
~~~~~~~~
import {
  STORIES_ADD,
# leanpub-start-insert
  STORIES_FETCH_ERROR,
# leanpub-end-insert
} from '../constants/actionTypes';

...

# leanpub-start-insert
const applyFetchErrorStories = (state, action) => ({
  stories: [],
  error: action.error,
});
# leanpub-end-insert

function storyReducer(state = INITIAL_STATE, action) {
  switch(action.type) {
    case STORIES_ADD : {
      return applyAddStories(state, action);
    }
# leanpub-start-insert
    case STORIES_FETCH_ERROR : {
      return applyFetchErrorStories(state, action);
    }
# leanpub-end-insert
    default : return state;
  }
}

export default storyReducer;
~~~~~~~~

In your story selector, you would have to change the structure of the story state. The story state isn't anymore a mere list of stories but a complex object with a list of stories and an error object. In addition, you could add a second selector to select the error object. It will be used later on in a component.

{title="src/selectors/story.js",lang="javascript"}
~~~~~~~~
...

const getReadableStories = ({ storyState, archiveState }) =>
# leanpub-start-insert
  storyState.stories.filter(isNotArchived(archiveState));
# leanpub-end-insert

# leanpub-start-insert
const getFetchError = ({ storyState }) =>
  storyState.error;
# leanpub-end-insert

export {
  getReadableStories,
# leanpub-start-insert
  getFetchError,
# leanpub-end-insert
};
~~~~~~~~

Last but not least, in your component you can retrieve the error object in your connect higher-order component and display with React's [conditional rendering](https://www.robinwieruch.de/conditional-rendering-react/) an error message in case of an error in the state.

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
...
# leanpub-start-insert
import {
  getReadableStories,
  getFetchError,
} from '../selectors/story';
# leanpub-end-insert

...

# leanpub-start-insert
const Stories = ({ stories, error }) =>
# leanpub-end-insert
  <div className="stories">
    <StoriesHeader columns={COLUMNS} />

# leanpub-start-insert
    { error && <p className="error">Something went wrong ...</p> }
# leanpub-end-insert

    {(stories || []).map(story =>
      ...
    )}
  </div>

...

const mapStateToProps = state => ({
  stories: getReadableStories(state),
# leanpub-start-insert
  error: getFetchError(state),
# leanpub-end-insert
});

...
~~~~~~~~

In your browser in the developer console, you can simulate being offline. You can try it and see that an error message shows up when searching for stories. When you go online again and search for stories, the error message should disappear. Instead a list of stories displays again. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/a1f6a885357a891b5e94ade90728a1f2d3d1dbb9).

### Part 17: Testing

Every application in production should be tested. Therefore, the next step could be to add a couple of tests to your application. The chapter will only cover a handful of tests to demonstrate testing in Redux. You could add more of them on your own. However, the chapter will not test your view layer, because this is covered in "The Road to learn React".

Since you have set up your application with create-react-app, it already comes with [Jest](https://facebook.github.io/jest/) to test your application. You can give a filename the prefix *test* to include it in your test suite. Once you run `npm test` on the command line, all included tests will get executed. The following files were not created for you, thus you would have to create them on your own.

First, let's create a test file for the story reducer. As you have learned, a reducer gets a previous state and an action as input and returns a new state. It is a pure function and thus it should be simple to test because it has no side-effects.

{title="src/reducers/story.test.js",lang="javascript"}
~~~~~~~~
import storyReducer from './story';

describe('story reducer', () => {
  it('adds stories to the story state', () => {
    const stories = ['a', 'b', 'c'];

    const action = {
      type: 'STORIES_ADD',
      stories,
    };

    const previousState = { stories: [], error: null };
    const expectedNewState = { stories, error: null };

    const newState = storyReducer(previousState, action);

    expect(newState).toEqual(expectedNewState);;
  });
});
~~~~~~~~

Basically you created the necessary inputs for your reducer and the expected output. Then you can compare both in your expectation. It depends on your test philosophy whether you create the action again in the file or import your action creator that you already have from your application. In this case, an action was used.

In order to verify that your previous state isn't mutated when creating the new state, because Redux embraces immutable data structures, you could use a neat helper library that freezes your state.

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save-dev deep-freeze
~~~~~~~~

In this case, it can be used to freeze the previous state.

{title="src/reducers/story.test.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import deepFreeze from 'deep-freeze';
# leanpub-end-insert
import storyReducer from './story';

describe('story reducer', () => {
  it('adds stories to the story state', () => {
    const stories = ['a', 'b', 'c'];

    const action = {
      type: 'STORIES_ADD',
      stories,
    };

    const previousState = { stories: [], error: null };
    const expectedNewState = { stories, error: null };

# leanpub-start-insert
    deepFreeze(previousState);
# leanpub-end-insert
    const newState = storyReducer(previousState, action);

    expect(newState).toEqual(expectedNewState);;
  });
});
~~~~~~~~

Now, every time you would mutate accidentally your previous state in the reducer, an error in your test would show up. It is up to you to add two more tests for the story reducer. One test could verify that an error object is set when an error occurs and another test that verifies that the error object is set to null when stories are successfully added to the state.

Second, you can add a test for your selectors. Let's demonstrate it with your story selector. Since the selector function is a pure function again, you can easily test it with an input and an expected output. You would have to define your global state and use the selector the retrieve an expected substate.

{title="src/selectors/story.test.js",lang="javascript"}
~~~~~~~~
import { getReadableStories } from './story';

describe('story selector', () => {
  it('retrieves readable stories', () => {
    const storyState = {
      error: null,
      stories: [
        { objectID: '1', title: 'foo' },
        { objectID: '2', title: 'bar' },
      ],
    };
    const archiveState = ['1'];
    const state = { storyState, archiveState }

    const expectedReadableStories = [{ objectID: '2', title: 'bar' }];
    const readableStories = getReadableStories(state);

    expect(readableStories).toEqual(expectedReadableStories);;
  });
});
~~~~~~~~

That's it. Your Redux state is a combination of the `storyState` and the `archiveState`. When both are defined, you already have your global state. The selector is used to retrieve a substate from the global state. Thus you would only have to check if all the readable stories that were not archived are retrieved by the selector.

Third, you can add a test for your action creators. An action creator only gets a payload and returns an action object. The expected action object can be tested.

{title="src/actions/story.test.js",lang="javascript"}
~~~~~~~~
import { doAddStories } from './story';

describe('story action', () => {
  it('adds stories', () => {
    const stories = ['a', 'b'];

    const expectedAction = {
      type: 'STORIES_ADD',
      stories,
    };
    const action = doAddStories(stories);

    expect(action).toEqual(expectedAction);;
  });
});
~~~~~~~~

As you can see, testing reducers, selectors and action creators follows always a similar pattern. Due to the functions being pure functions, you can focus on the input and output of these functions. In the previous examples all three test cases were strictly decoupled. However, you could also decide to import your action creator in your reducer test avoid creating a hard coded action. You can find this part of the chapter in [the GitHub repository](https://github.com/rwieruch/react-redux-hackernews/tree/d1fcb31b7a1b1602069718941844d08c21583607).

### Final Words

Implementing this application could go on infinitely. I would have plenty of features in my head that I would want to add to it. What about you? Can you imagine to continue building this application? From a technical perspective, things that were taught in this book, everything is set up to give you the perfect starting point. However, there were more topics in this book that you could apply. For instance, you could normalize your incoming stories from the API before they reach the Redux store. The following list should give you an idea about potential next steps:

* Normalization: The data that comes from the Hacker News API could be normalized before it reaches the reducer and finally the Redux store. You could use the normalizr library that was introduced earlier in the book. It might be not necessary yet to normalize your state, but in a growing application you would normalize your data eventually. The data would be normalized between fetching the data and sending it via an action creator to the reducers.

* Local State: So far you have only used Redux. But what about mixing local state into the application? Could you imagine a use case for it? For instance, you would be able to distinguish between readable and archived stories in your application. There could be a toggle, that is true or false in your `Stories` component as local state, that decides whether the component shows readable or archived stories. Depending on the toggle in your view layer, you would retrieve either readable or archived stories via selectors from your Redux store and display them.

* React Router: Similar to the previous step, using a toggle to show archived and readable stories, you could add a view layer Router to display these different stories on two routes. It could be React Router when using React as your view layer. All of this is possible, because fortunately you don't delete stories when archiving them from your Redux store, but keep a list of archived stories in a separate substate.

* Paginated Data: The response from the Hacker News API doesn't only return the list of stories. It returns a paginated list of stories with a page property. You could use the page property to fetch more stories with the same search term. The list component in React could be a [paginated list](https://www.robinwieruch.de/react-paginated-list/) or [infinite scroll list](https://www.robinwieruch.de/react-infinite-scroll/).

* Caching: You could cache the incoming data from the Hacker News API in your Redux store. It could be cached by search term. When you search for a search term twice, the Redux store could be used, when a result by search term is already in place. Otherwise a request to the Hacker News API would be made. In [the Road to learn React](https://www.robinwieruch.de/the-road-to-learn-react/) readers create a cache in React's local state. However, the same can be done in a Redux store.

* Local Storage: You already keep track of your archived stories in the Redux store. You could introduce the native local storage of the browser, as you have seen in the plain React chapters, to keep this state persistent. When a user loads the application, there could be a lookup in the local storage for archived stories. If there are archived stories, they could be rehydrated into the Redux store. When a story gets archived, it would be dehydrated into the local storage too. That way you would keep the list of archived stories in your Redux store and local storage in sync, but would add a persistent layer to it when an user closes your application and comes back later to it.

As you can see, there are a multitude of features you could implement or techniques you could make use of. Be curious and apply these on your own. After you come up with your own implementations, I am keen to see them. Feel free to reach out to me on [Twitter](https://twitter.com/rwieruch).
