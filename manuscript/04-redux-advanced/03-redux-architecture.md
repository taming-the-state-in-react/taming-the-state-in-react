## Redux State as Architecture

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

You have learned that you can plan your state management ahead. There are use cases where local state makes more sense than sophisticated state. Both can be used and should be used in a scaling application. By combining local state to the native local storage of the browser, you can give the user of your application an improved UX. In addition, you can plan the state ahead too. Think about view state and entitiy state and where it should live in your application. You can give your reducers differenct domains as their ownership such as todoReducer, filterReducer and notificationReducer. However, once you have planned your state management and state, don't stick to it. When slaing your application, always revisit those things to apply refactorings. That will help you to keep your state manageable, maintainable and predictable in the long run.

## Hands On: Hacker News with Redux

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

### Part 1: Project Organization

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
mkdir constants reducers actions components store
~~~~~~~~

- your folder structure should be similiar to the following:

{title="Folder Structure",lang="text"}
~~~~~~~~
-src/
--actions/
--components/
--constants/
--reducers/
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
---index.js
--components/
---App.js
---App.css
---Stories.js
---Stories.css
---Story.js
---Story.css
--constants/
---index.js
---actionTypes.js
--reducers/
---index.js
--store/
---index.js
---store.js
--index.css
--index.js
~~~~~~~~

- the *index.js* files are used as entry point in each folder, that's a personal opionated constraint that I am used to do in order to keep the folders encapsulated as modules, only the *index.js* gives access to the folder

- now you have your foundation of folders and files for your React and Redux application, except for the specific component files that you already have, everything else can be used as a blueprint for any application using React and Redux

### Part 2: Project Organization

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

### Part 3: Styling

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
  dismiss: {
    width: '10%',
  },
};
# leanpub-end-insert

class Stories extends Component {
  ...
}
~~~~~~~~

- the last column with the 'dismiss' property name is not used yet
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
        <span style={{ width: columns.dismiss.width }}>
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

### Part 4: Dismiss a Story

- in this part you add your first functionality: dismissing a story
- therefore you will introduce Redux to your application
- it would be possible in plain React too, but since it is a Redux application, you will use its functionalities rather than the plain React state management

- first, the functionality can be passed down to the `Story` component from above
- in the beginning it can be an empty function, later on it will get wired up to Redux

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
class App extends Component {
  render() {
    return (
      <div className="app">
        <Stories
          stories={this.props.stories}
# leanpub-start-insert
          onDismiss={() => {}}
# leanpub-end-insert
        />
      </div>
    );
  }
}
~~~~~~~~

- second, you can pass it through your `Stories` component, you might already notice that this could be a potential refactor later on, because the function gets passed from the root component through a few components only to reach the leaf component

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
class Stories extends Component {
  render() {
    const {
      stories,
# leanpub-start-insert
      onDismiss,
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
            onDismiss={onDismiss}
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
      onDismiss,
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
        <span style={{ width: columns.dismiss.width }}>
# leanpub-start-insert
          <button
            type="button"
            className="button-inline"
            onClick={() => onDismiss(objectID)}
          >
            Dismiss
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
        <span style={{ width: columns.dismiss.width }}>
# leanpub-start-insert
          <ButtonInline onClick={() => onDismiss(objectID)}>
            Dismiss
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

### Part 5: Introduce Redux

- this part will introduce Redux to manage the state of the (sample) stories, the first action will be the dismissing of a story from the list
- but let's approach this step by step

{title="Command Line",lang="text"}
~~~~~~~~
npm install --save redux
~~~~~~~~

- action creator
- reducer
- store
- naive wire up

### Part 6: Wire Up React with Redux

- sophistaicated wire up