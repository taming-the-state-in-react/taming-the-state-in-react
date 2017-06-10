# Redux State as Architecture

The book taught you the practical usage of Redux. You have learned about the main parts in the Redux state management architecture: actions, reducers and the store.

{title="Code Playground",lang="javascript"}
~~~~~~~~
Action -> Reducer(s) -> Store
~~~~~~~~

The chain is connected to the view by something (e.g. react-redux with `mapStateToProps()` and `mapDispatchToProps()`) that enables you to write connected components. These components have access to the Redux store. They are used to receive state or to alter the state. They are a specialized case of a container component in the presenter and container pattern when using components.

{title="Code Playground",lang="javascript"}
~~~~~~~~
View -> (mapDispatchToProps) -> Action -> Reducer(s) -> Store -> (mapStateToProps) -> View
~~~~~~~~

All other components are not aware of any local or sophisticated state management solution. They only receive props, except they have their own local state management (such as `this.state` and `this.setState()` in React).

State can be received directly by operating on the state object or indirectly by selecting it with selectors.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// directly from state object
state.something;

// indirectly from state object via selector
const getSomething = (state) => state.something;
~~~~~~~~

State can be altered by dispatching an action directly or by using action creators that return an action object.

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

In order to keep your state predictable and manageable in the reducers, you can apply techniques for an improved state structure. You can normalize your state to have always a single source of truth. That means you don't have to operate on duplicated entities, but only on one reference of the entity. In addition, it keeps the state flat. It is easier to manage only by using spread operators.

Around these practical usages, you have learned several supporting techniques. There are tons of opinionated ways to organize your folders and files. The book showcased two of the main approaches, but they vary in their execution from developer to developer, team to team or company to company. Nevertheless, you should always bear in mind to keep Redux at a top level. It is not used to manage the state of one single component. Instead it is used to wire dedicated components to the store in order to enable them to alter and to retrieve the state from it.

Coupling actions and reducers is fine, but always think twice when adding another action type. For instance, perhaps a action type could be reused in another reducer. When reusing action types, you avoid to end up with fat thunks when using Redux thunk. Instead of dispatching several actions, your thunk could dispatch only one abstract action that is reused in more than one reducer.

You have learned that you can plan your state management ahead. There are use cases where local state makes more sense than sophisticated state. Both can be used and should be used in a scaling application. By combining local state to the native local storage of the browser, you can give the user of your application an improved UX. In addition, you can plan the state ahead too. Think about view state and entitiy state and where it should live in your application. You can give your reducers differenct domains as their ownership such as `todoReducer`, filterReducer and notificationReducer. However, once you have planned your state management and state, don't stick to it. When slaing your application, always revisit those things to apply refactorings. That will help you to keep your state manageable, maintainable and predictable in the long run.

# Hands On: Hacker News with Redux

{title="Command Line",lang="text"}
~~~~~~~~
create-react-app react-redux-hackernews
~~~~~~~~

- navigate in your application folder, open your editor in it and optionally start the application

{title="Command Line",lang="text"}
~~~~~~~~
cd react-redux-hackernews
npm start
~~~~~~~~

## Part 1: Project Organization

- part 1: preparing the app folder structure
- move into the *src/* folder and delete the boilerplate files

{title="Command Line: /",lang="text"}
~~~~~~~~
cd src
rm logo.svg App.js App.test.js App.css
~~~~~~~~

- from the *src/* folder create the folders for a organized folder structure by technical separation

{title="Command Line: src/",lang="text"}
~~~~~~~~
mkdir constants reducers actions selectors sagas components store
~~~~~~~~

- your folder structure should be similiar to the following:

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

- navigate in the *component/* folder and create the following files for independent components

{title="Command Line: src/",lang="text"}
~~~~~~~~
cd components
touch index.js App.js Stories.js Story.js
~~~~~~~~

- continue this way and create the remaining files to end up with the following folder structure

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
---sagas/
----index.js
--selectors/
--store/
---index.js
--index.css
--index.js
~~~~~~~~

- now you have your foundation of folders and files for your React and Redux application, except for the specific component files that you already have, everything else can be used as a blueprint for any application using React and Redux

## Part 2: Plain React Components

- part 2: build react component architecture that only receives all the necessary props from above, props can already have callback functions to do something, unawre of doing it in the state or somewhere else

- in your entry point to React, where your root component gets rendered into the DOM, adjust the import of the `App` component

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

- in the next step, let's come up with sample data that can be used in the React components, the sample data gets defined and becomes the input of the `App` component, later this data will get fetched from the Hacker News API

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components';
import './index.css';

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

ReactDOM.render(<App stories={stories} />, document.getElementById('root'));
# leanpub-end-insert
~~~~~~~~

- the three components, `App`, `Stories` and `Story`, are not defined yet, let's define them component by component,
- first, the `App` receives the sample stories from above as props and its only responsiblity is to render the `Stories` component and to pass over the `stories` as props

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import './App.css';

import Stories from './Stories';

class App extends Component {
  render() {
    return (
      <div className="app">
        <Stories stories={this.props.stories} />
      </div>
    );
  }
}

export default App;
~~~~~~~~

- second, the `Stories` component receives the `stories` as props and renders for each story a `Story` component

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import './Stories.css';

import Story from './Story';

class Stories extends Component {
  const { stories } = this.props;

  render() {
    return (
      <div className="stories">
        {(stories || []).map(story =>
          <Story
            key={story.objectID}
            story={story}
          />
        )}
      </div>
    );
  }
}

export default Stories;
~~~~~~~~

- third, the `Story` component renders a few properties of the `story` object that comes in as props

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import './Story.css';

class Story extends Component {
  render() {
    const {
      story,
    } = this.props;

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
}

export default Story;
~~~~~~~~

- now you can start your application again with `npm start`
- you should see both sample stories displayed in plain React

## Part 3: Apply Styling

- the application looks a bit dull, therefore we will quickly drop in styling
- first, the application needs some general style that can be defined in the root style file

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

- second, the `App` component gets style

{title="src/components/App.css",lang="css"}
~~~~~~~~
.app {
  margin: 20px;
}

.interactions {
  text-align: center;
}
~~~~~~~~

- third, the `Stories` component gets style

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

- and last but not least, the `Story` component gets style

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

- when you start your application again, it seems more organized, but there is still something missing
- the columns for each story should be aligned and perhaps there should be a heading for each column
- first, you can define an object to describe the columns

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
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

class Stories extends Component {
  ...
}
~~~~~~~~

- the last column with the 'archive' property name is not used yet
- second, you can pass this object to your `Story` component

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
class Stories extends Component {
  render() {
    const { stories } = this.props;

    return (
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
    );
  }
}
~~~~~~~~

- the story component can use it to style each entry

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
class Story extends Component {
  render() {
    const {
      story,
# leanpub-start-insert
      columns,
# leanpub-end-insert
    } = this.props;

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
}
~~~~~~~~

- last but not least, you can use the `COLUMN` object to give your stories a header columns
- rather than doing it manually as in the `Story` component, let's map the object dynamically to render it, since it is an object, you have to turn it into an array of the property names first, and then access the object by its mapped keys

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
class Stories extends Component {
  render() {
    const { stories } = this.props;

    return (
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
    );
  }
}
~~~~~~~~

- before you start your application, you can extract the header columns as own component

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
class Stories extends Component {
  render() {
    const { stories } = this.props;

    return (
      <div className="stories">
# leanpub-start-insert
        <StoriesHeader columns={COLUMNS} />
# leanpub-end-insert

        {(stories || []).map(story =>
          <Story
            key={story.objectID}
            story={story}
            columns={COLUMNS}
          />
        )}
      </div>
    );
  }
}

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

- the rendering of the sample stories is done and is representable to none designers

## Part 4: Archive a Story

- in this part you add your first functionality: archiving a story
- therefore you will introduce Redux to your application
- it would be possible in plain React too, but since it is a Redux application, you will use its functionalities rather than the plain React state management

- first, the functionality can be passed down to the `Story` component from above
- in the beginning it can be an empty function, later on it will get wired up to Redux

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

- second, you can pass it through your `App` and `Stories` component
- you might already notice that this could be a potential refactor later on, because the function gets passed from the root component through a few components only to reach the leaf component

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  render() {
    return (
      <div className="app">
        <Stories
          stories={this.props.stories}
# leanpub-start-insert
          onArchive={this.props.onArchive}
# leanpub-end-insert
        />
      </div>
    );
  }
}
~~~~~~~~

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
class Stories extends Component {
  render() {
    const {
      stories,
# leanpub-start-insert
      onArchive,
# leanpub-end-insert
    } = this.props;

    return (
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
    );
  }
}
~~~~~~~~

- third, you can use it in your `Story` component in a button

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
class Story extends Component {
  render() {
    const {
      story,
      columns,
# leanpub-start-insert
      onArchive,
# leanpub-end-insert
    } = this.props;

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
}
~~~~~~~~

- a great refactoring would be to extract the button as reusable component

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
class Story extends Component {
  render() {
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

- you can make it another more abstract Button component that doesn't share the inline style

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
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

- both buttons should be extracted to a new file called *src/components/Buttons.js* but exported that at least the `ButtonInline` component can get reused in the `Story` component, that's up to you

- now, when you start your application, the button is there, but it doesn't work because it only receives an no-op (empty function) as property

## Part 5: Introduce Redux: Store + First Reducer

- this part will introduce Redux to manage the state of the (sample) stories instead of passing it directly into your component tree
- let's approach this step by step
- first, you have to install Redux on the command line.

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux
~~~~~~~~

- second, in the root entry point of React, you can import the Redux store, the store is not yet defined
- instead of using the sample stories, you use the stories that are stored in the Redux store
- assuming the store saves only a list of stories as state, you can simply get the root state of the store and assume that it is the list of stories

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
  <App stories={store.getState()} />,
# leanpub-end-insert
  document.getElementById('root')
);
~~~~~~~~

- third, you have to create your Redux store instance, it already takes a reducer that is not implemented yet, you will implement it in the next step

{title="src/store/index.js",lang="javascript"}
~~~~~~~~
import { createStore } from 'redux';
import storyReducer from '../reducers/story';

const store = createStore(
  storyReducer
);

export default store;
~~~~~~~~

- fourth, in your *src/reducers/* folder you can create your first reducer: `storyReducer`
- as initial state it can have the sample stories

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

- your application should work when you start it
- it is using the Redux store to retrieve the initial state from the `storyReducer`, because it is the only reducer by now in your application
- there are no actions yet and no action is captured in the reducer yet
- even though there was no action dispatched yet, you can see that the Redux store runs through all its defined reducers to initialize its initial state in the store

## Part 7: Two Reducers

- you have used the Redux store to save an initial state of sample stories and to retrieve this state for your component tree
- but there is no state manipulation happening yet
- in this part and the next part you are going to implement the archive function
- when approaching this functionality, the simplest thing to do would be to remove the archived story from the list of stories in the `storyReducer`
- but let's approach this from a different angle to have a greater impact in the long run
- it could still be useful to have all stories in the end, but have a way to distinguish between them: stories and archived stories
- following this way, you would be able in the future to have a second component that shows the archived stories

- thus the `storyReducer` will stay as it is for now
- but you will have to introduce a second reducer, a `archiveReducer`, that keeps a list of references to the archived stories

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

- you will implement the action to dismiss a story in a second
- but the Redux store in its instantation needs to get both reducers now, it has to get the combined reducer
- let's pretend that the store can import the combined reducer from the entry file, the *reducers/index.js* without worrying about the combining of the reducers

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

- now you can combine both reducers

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

- since your state is sliced up into two substates now, you have to adjust how you retrieve the stories from your store with the intermediate `storyState`
- this is a crucial steo, because it shows how a combined reducer slices up your state into substates

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

- the application should show up the same as before when you start it
- finally in the next part you will dispatch your first action to dismiss a story

## Part 8: First Action

- the archive action needs to be captured in the `archiveReducer`, it simply stores all archived stories by their id in a list

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

- it uses a constant action type from a different file
- it is already defined in another file to be reused when dispatching the action

{title="src/constants/actionTypes.js",lang="javascript"}
~~~~~~~~
export const STORY_ARCHIVE = 'STORY_ARCHIVE';
~~~~~~~~

- last but not least, you can dispatch the action in your root component

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

- you dispatch the action directly without an action creator
- when you start your application, it shoudl still work, but nothing happens when archiving a story
- the archived stories are not used yet in the component tree

## Part 9: First Selector

- you can use both substates, `storyState` and `archiveState` to compute the not lsit of stories that are not dismissed
- the deriving of those properties can happen in a selector

- create your first selector that only returns the part of the stories that is not archived

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

- the selector makes heaviliy use of JavaScript ES6 arrow functions, JavaScript ES6 destructuring and a higher order function: isNotArchived()`
- don't feel intimidated by it, it is only a way to express these functions more concise in a functional programmign style
- in plain JavaScript ES5 it would look like the following:

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

- now you can use the selector instead of retrieving all stories from the store directly

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

- still, nothing happens when you archive a story

## Part 11: Re-render View

- when an action dispatches, the state in the Redux store gets updated
- however, the component tree in React doesn't update
- no one subscribed to the Redux store yet
- in the first attempt, you will wire up Redux and React naively and re-render the whole component tree on each update

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

- now the components will re-render once you archive a story
- congratulations, you dispatched your first action, selected derived properties from the state and updated your component tree by subscribing it to the Redux store

## Part 12: First Middleware

- often you don't notice when an action is dispatched
- therefore you can use the [redux-logger](https://github.com/evgenyrodionov/redux-logger) middleware in your Redux store to `console.log()` every action, previous state and next state automarically to your developers console when dispatching an action
- first, install the neat middleware library

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux-logger
~~~~~~~~

- now, use it as middleware in your Redux store initialization

{title="src/store.js",lang="javascript"}
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

- every time you dispatch an action now, for instance when archiving a story, you will see the logging in the developer console in your browser

## Part 13: First Action Creator

- you dispatch already an action without using an action creator
- action creators are not mandatory, but they keep your Redux architecture organized
- the action that gets directly dispatched can be refactored as action creator
- first, you can define the action that takes an story id to archive a story in a new file

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

- second, you can use it in your root component
- instead of dispatchign the action object directly, you can create an action object by using its action creator

{title="src/actions/archive.js",lang="javascript"}
~~~~~~~~
...
import { doArchiveStory } from './actions/archive';

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

- the application should work as before

## Part 14: Connect React with Redux

- the component tree already re-renders when you dispatch an action
- but you want to wire up component indepdently with the Redux store without using the Redux store directly
- moreover you don't want to re-render the whole component tree, but only the components where the state or props have changed
- first, install the library that can be used to connect both worlds

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save react-redux
~~~~~~~~

- use the `Provider` component, that makes the Redux store available to all component below, in your root component

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

- notice that the render method isn't re-render anymore, no one subscribes to the Redux store and the `App` component isn't receiving any props
- instead the `App` is only rendering component and doesn't pass any props anymore

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import './App.css';

import Stories from './Stories';

class App extends Component {
  render() {
    return (
      <div className="app">
# leanpub-start-insert
        <Stories />
# leanpub-end-insert
      </div>
    );
  }
}

export default App;
~~~~~~~~

- but who gives the props to the `Stories` component?
- this component needs to know at least about the list of stories, because it has to map over it
- the solution is to upgrade the `Stories` component to a connected component
- instead of only default exporting the plain `Stories` component:

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
...

export default Stories;
~~~~~~~~

- you can export the connected component that has access to the store

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

- the `Stories` component is a connected component now and is the only component that has access to the Redux store

## Part 15: Lift Connected Components

- you can give more components access to the Redux store by transforming a component to a connected component
- the archive functionality is connected in the `Stories` component, but it is only used in the `Story` component
- you can remove this functionality from the `Stories`component and don't pass the `onArchive()` function anymore to the `Story` component

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
...

class Stories extends Component {
  render() {
    const { stories } = this.props;

    return (
      <div className="stories">
        <StoriesHeader columns={COLUMNS} />

        {(stories || []).map(story =>
          <Story
            key={story.objectID}
            story={story}
            columns={COLUMNS}
          />
        )}
      </div>
    );
  }
}

...

const mapStateToProps = state => ({
  stories: getReadableStories(state),
});

export default connect(
  mapStateToProps
)(Stories);
~~~~~~~~

- instead you can connect the `Story` component

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

## Part 16: Interacting with an API

- writing applications with sample data is dull
- it can be more exciting by interacting with a real API - the [Hacker News API](https://hn.algolia.com/api)
- even though, as you know, you can have asynchronous actions without another library, this application will introduce Redux saga as asynchrnours action library

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux-saga
~~~~~~~~

- first, introduce a root saga, it can be similar seen to the combined root reducer, in the end the Redux store expects one reducer or one saga for its saga middleware

{title="src/sagas/index.js",lang="javascript"}
~~~~~~~~
import { takeEvery } from 'redux-saga/effects';
import { STORIES_FETCH } from '../constants/actionTypes';
import { handleFetchStories } from './story';

function *watchAll() {
  yield [
    takeEvery(STORIES_FETCH, handleFetchStories),
  ]
}

export default watchAll;
~~~~~~~~

- second, the root saga can be used in the Redux store middleware when initializing the saga middleware

{title="src/sagas/index.js",lang="javascript"}
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

- third, introduce the new action type in your constants that triggers the saga, but already introduce a second action type that will later on, when the request succeeds, add the stories in your `storyReducer`

{title="src/constants/actionTypes.js",lang="javascript"}
~~~~~~~~
export const STORY_ARCHIVE = 'STORY_ARCHIVE';
# leanpub-start-insert
export const STORIES_FETCH = 'STORIES_FETCH';
export const STORIES_ADD = 'STORIES_ADD';
# leanpub-end-insert
~~~~~~~~

- and fourth implement the story saga that encapsulates the API request

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

- fourth, you need to define both actions creators: the first one that triggers the side-effect to fetch stories by a search term and the second one to add the fetched stories to your store

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

- the second action needs to be intercepted in your `storyReducer` to store the stories, don't forget to remove the sample stories

{title="src/reducers/story.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { STORIES_ADD } from '../constants/actionTypes';

const INITIAL_STATE = [];

const applyAddStories = (state, action) =>
  [ ...action.stories ];
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

- everything is setup from a Redux and Redux Saga perspective
- now only one component from the view layer needs to trigger the `STORIES_FETCH` action that is intercepted in the saga, fetches the stories and stores them in the Redux store with the `STORIES_ADD` action

- in your `App` component you can introduce the new `SearchStories` Component

{title="src/reducers/story.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import './App.css';

import Stories from './Stories';
# leanpub-start-insert
import SearchStories from './SearchStories';
# leanpub-end-insert

class App extends Component {
  render() {
    return (
      <div className="app">
# leanpub-start-insert
        <div className="interactions">
          <SearchStories />
        </div>
# leanpub-end-insert
        <Stories />
      </div>
    );
  }
}

export default App;
~~~~~~~~

- the `SearchStories` component is a connected component, the next step is to implement that component
- first, you start with a plain React component that has a form, input field and button

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import { Button } from './Buttons';

const applyQueryState = query => () => ({
  query
});

class SearchStories extends Component {
  constructor(props) {
    super(props);

    this.state = {
      query: '',
    };

    this.onChange = this.onChange.bind(this);
    this.onSubmit = this.onSubmit.bind(this);
  }

  onSubmit(event) {
    const { query } = this.state;
    if (query) {
      // TODO: not defined yet
      this.props.onFetchStories(query)

      this.setState(applyQueryState(''));
    }

    event.preventDefault();
  }

  onChange(event) {
    const { value } = event.target;
    this.setState(applyQueryState(value));
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

- the component receives a function from the props
- this function will be defined in the connected component to dispatch the action

- second, you have to connect the component to make the dispatch functionality available

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import { connect } from 'react-redux';
import { doFetchStories } from '../actions/story';
# leanpub-end-insert
import { Button } from './Buttons';

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

- the search should work now
- a last refactoring step could be to separate saga and api

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

## Final Words

- features you could add
- normalizing the state before it reaches the reducer with normalizr
- introduce react router to have two routes: search and archived
- paginated fetch
- error handling
- tests
- archived in local storage
- caching