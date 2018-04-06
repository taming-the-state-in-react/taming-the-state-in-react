## Hands On: Hacker News with MobX

In this chapter, you will be guided to build your own [Hacker News](https://news.ycombinator.com/) application with React and MobX. Hacker News is a platform to share news in and around the technology domain. It provides a [public API](https://hn.algolia.com/api) to retrieve their data.

You are going to use create-react-app to bootstrap your project. You can read the [official documentation](https://github.com/facebookincubator/create-react-app) to get to know how it works. After you have installed it, you simply start by choosing a project name for your application.

{title="Command Line",lang="text"}
~~~~~~~~
create-react-app react-mobx-hackernews
~~~~~~~~

After the project was created for you, you can navigate into the project folder, open your editor and start the application.

{title="Command Line",lang="text"}
~~~~~~~~
cd react-mobx-hackernews
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

Even the `App` component is removed, because you'll organize it in folders instead of in the top level *src/* folder. Now, from the *src/* folder, create the folders for an organized folder structure by a technical separation.

{title="Command Line: src/",lang="text"}
~~~~~~~~
mkdir components stores api
~~~~~~~~

Your folder structure should be similar to the following:

{title="Folder Structure",lang="text"}
~~~~~~~~
-src/
--api/
--components/
--stores/
--index.css
--index.js
~~~~~~~~

Navigate in the *components/* folder and create the following files for your independent components. These are not all components yet. You will create more of them on your own for this application.

{title="Command Line: src/",lang="text"}
~~~~~~~~
cd components
touch App.js Stories.js Story.js App.css Stories.css Story.css
~~~~~~~~

You can continue this way and create the remaining files to end up with the following folder structure.

{title="Folder Structure",lang="text"}
~~~~~~~~
-src/
--api/
--components/
---App.js
---App.css
---Stories.js
---Stories.css
---Story.js
---Story.css
--stores/
---index.js
--index.css
--index.js
~~~~~~~~

Now you have your foundation of folders and files for your React and MobX application. Except for the specific component files that you already have, everything else can be used as a blueprint, your own boilerplate, for any application using React and MobX. But only if it is separated by technical concerns. In a growing application, you might want to separate your folders by feature.

### Part 2: Plain React Components

### Part 3: Apply Styling

### Part 4: Archive a Story

### Part 5: Getting Decorators to Work in CRA

1) Run create-react-app. This creates a new app with the official configuration.

2) Run npm run eject. This moves files around and makes your app’s configuration accessible.

3) Run npm install --save-dev babel-plugin-transform-decorators-legacy. This installs the Babel plugin for decorators. It’s called legacy even though it’s a feature from the far future.

4) Open package.json, find the "babel" section (line 78 for me), and add 4 lines so it looks like this:

"babel": {
  "plugins": [
    "transform-decorators-legacy"
  ],
  "presets": [
    "react-app"
  ]
},

### Part 6: Introduce MobX: Store

cd src/stores
touch storyStore.js

npm install --save mobx

{title="src/stores/storyStore.js",lang="javascript"}
~~~~~~~~
import { observable } from 'mobx';

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

class StoryStore {
  @observable stories = INITIAL_STATE;
}

const storyStore = new StoryStore();

export default storyStore;
~~~~~~~~

- export an instance of store
- initial state can be passed in constructor, but also directly allocated in store

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
# leanpub-start-insert
import storyStore from './stores/storyStore';
# leanpub-end-insert
import './index.css';

ReactDOM.render(
# leanpub-start-insert
  <App stories={storyStore.stories} onArchive={() => {}} />,
# leanpub-end-insert
  document.getElementById('root')
);
~~~~~~~~

### Part 7: Second Store

- second store to capture the archived stories

touch archiveStore

{title="src/stores/archiveStore.js",lang="javascript"}
~~~~~~~~
import { observable } from 'mobx';

class ArchiveStore {
  @observable archivedStoryIds = [];
}

const archiveStore = new ArchiveStore();

export default archiveStore;
~~~~~~~~

- constructor is not mandatory
- mutate directlz on the store

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
import storyStore from './stores/storyStore';
# leanpub-start-insert
import archiveStore from './stores/archiveStore';
# leanpub-end-insert
import './index.css';
import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(
  <App
    stories={storyStore.stories}
# leanpub-start-insert
    onArchive={id => archiveStore.archivedStoryIds.push(id)}
# leanpub-end-insert
  />,
  document.getElementById('root')
);
~~~~~~~~

### Part 8: Combined Stores for Computed Readable Stories

- stories and archived stories are available in two separte stores
- could compute the readable stories in a react component, but wouldnt it be better to already compoute these in a store?
- the storyStore needs somehow access to the archiveStore to compute readable stories

{title="src/stores/index.js",lang="javascript"}
~~~~~~~~
import StoryStore from './storyStore';
import ArchiveStore from './archiveStore';

class RootStore {
  constructor() {
    this.storyStore = new StoryStore(this);
    this.archiveStore = new ArchiveStore(this);
  }
}

const rootStore = new RootStore();

export default rootStore;
~~~~~~~~

{title="src/stores/storyStore.js",lang="javascript"}
~~~~~~~~
...

class StoryStore {
  @observable stories = INITIAL_STATE;

# leanpub-start-insert
  constructor(rootStore) {
    this.rootStore = rootStore;
  }
# leanpub-end-insert
}

# leanpub-start-insert
export default StoryStore;
# leanpub-end-insert
~~~~~~~~

{title="src/stores/archiveStore.js",lang="javascript"}
~~~~~~~~
...

class ArchiveStore {
  @observable archivedStoryIds = [];

# leanpub-start-insert
  constructor(rootStore) {
    this.rootStore = rootStore;
  }
# leanpub-end-insert
}

# leanpub-start-insert
export default ArchiveStore;
# leanpub-end-insert
~~~~~~~~

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import App from './components/App';
# leanpub-start-insert
import store from './stores';
# leanpub-end-insert
import './index.css';

ReactDOM.render(
  <App
# leanpub-start-insert
    stories={store.storyStore.stories}
    onArchive={id => store.archiveStore.archivedStoryIds.push(id)}
# leanpub-end-insert
  />,
  document.getElementById('root')
);
~~~~~~~~

{title="src/stores/storyStore.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { observable, computed } from 'mobx';
# leanpub-end-insert

...

# leanpub-start-insert
const isNotArchived = archivedStoryIds => story =>
  archivedStoryIds.indexOf(story.objectID) === -1;
# leanpub-end-insert

class StoryStore {
  @observable stories = INITIAL_STATE;

  constructor(rootStore) {
    this.rootStore = rootStore;
  }

# leanpub-start-insert
  @computed get readableStories() {
    const { archivedStoryIds } = this.rootStore.archiveStore;
    return this.stories.filter(isNotArchived(archivedStoryIds));
  }
# leanpub-end-insert
}

export default StoryStore;
~~~~~~~~

{title="src/index.js",lang="javascript"}
~~~~~~~~
...

ReactDOM.render(
  <App
# leanpub-start-insert
    stories={store.storyStore.readableStories}
# leanpub-end-insert
    onArchive={id => store.archiveStore.archivedStoryIds.push(id)}
  />,
  document.getElementById('root')
);
~~~~~~~~

### Part 9: Re-render View

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import { autorun } from 'mobx';
# leanpub-end-insert
import App from './components/App';
import store from './stores';
import './index.css';

# leanpub-start-insert
function render() {
# leanpub-end-insert
  ReactDOM.render(
    <App
      stories={store.storyStore.readableStories}
      onArchive={id => store.archiveStore.archivedStoryIds.push(id)}
    />,
    document.getElementById('root')
  );
# leanpub-start-insert
}
# leanpub-end-insert

# leanpub-start-insert
autorun(render);
# leanpub-end-insert
~~~~~~~~

### Part 10: Enforced Actions

- use confgiuration, enforce actions, be opinionated, no direct mutations on stores

{title="src/stores/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { configure } from 'mobx';
# leanpub-end-insert

import StoryStore from './storyStore';
import ArchiveStore from './archiveStore';

# leanpub-start-insert
configure({ enforceActions: true });
# leanpub-end-insert

class RootStore {
  constructor() {
    this.storyStore = new StoryStore(this);
    this.archiveStore = new ArchiveStore(this);
  }
}

const rootStore = new RootStore();

export default rootStore;
~~~~~~~~

- archiving shouldnt work anymore by mutation directly the state in the arcgive store

{title="src/stores/archiveStore.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { observable, action } from 'mobx';
# leanpub-end-insert

class ArchiveStore {
  @observable archivedStoryIds = [];

  constructor(rootStore) {
    this.rootStore = rootStore;
  }

# leanpub-start-insert
  @action archiveStory = id =>
    this.archivedStoryIds.push(id);
# leanpub-end-insert
}

export default ArchiveStore;
~~~~~~~~

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { autorun } from 'mobx';
import App from './components/App';
import store from './stores';
import './index.css';

function render() {
  ReactDOM.render(
    <App
      stories={store.storyStore.readableStories}
# leanpub-start-insert
      onArchive={id => store.archiveStore.archiveStory(id)}
# leanpub-end-insert
    />,
    document.getElementById('root')
  );
}

autorun(render);
~~~~~~~~

- now it works as before, but by enforcing explicit actions
- it is good to be opinionated to form best practices

### Part 11: Sophisticated React and MobX Connection

npm install --save mobx-react

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { autorun } from 'mobx';
# leanpub-start-insert
import { Provider } from 'mobx-react';
# leanpub-end-insert
import App from './components/App';
import store from './stores';
import './index.css';

function render() {
  ReactDOM.render(
# leanpub-start-insert
    <Provider archiveStore={store.archiveStore} storyStore={store.storyStore}>
      <App />
    </Provider>,
# leanpub-end-insert
    document.getElementById('root')
  );
}

autorun(render);
~~~~~~~~

{title="src/components/App.js",lang="javascript"}
~~~~~~~~
...

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

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import { inject } from 'mobx-react';
# leanpub-end-insert
import './Stories.css';
import Story from './Story';

...

# leanpub-start-insert
const Stories = ({ storyStore, archiveStore }) =>
# leanpub-end-insert
  <div className="stories">
    <StoriesHeader columns={COLUMNS} />

# leanpub-start-insert
    {(storyStore.readableStories || []).map(story =>
# leanpub-end-insert
      <Story
        key={story.objectID}
        story={story}
        columns={COLUMNS}
# leanpub-start-insert
        onArchive={archiveStore.archiveStory}
# leanpub-end-insert
      />
    )}
  </div>

...

# leanpub-start-insert
export default inject('storyStore', 'archiveStore')(Stories);
# leanpub-end-insert
~~~~~~~~

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import { inject, observer } from 'mobx-react';
# leanpub-end-insert
import './Stories.css';
import Story from './Story';

...

const Stories = ({ storyStore, archiveStore }) =>
  <div className="stories">
    <StoriesHeader columns={COLUMNS} />

    {(storyStore.readableStories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
        columns={COLUMNS}
        onArchive={archiveStore.archiveStory}
      />
    )}
  </div>

...

# leanpub-start-insert
export default inject('storyStore', 'archiveStore')(observer(Stories));
# leanpub-end-insert
~~~~~~~~

- remove autorun functionality

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'mobx-react';
import App from './components/App';
import store from './stores';
import './index.css';

ReactDOM.render(
# leanpub-start-insert
  <Provider { ...store }>
# leanpub-end-insert
    <App />
  </Provider>,
  document.getElementById('root')
);
~~~~~~~~

### Part 12: Injections and Observers Everywhere

{title="src/components/Story.js",lang="javascript"}
~~~~~~~~
import React from 'react';
# leanpub-start-insert
import { inject, observer } from 'mobx-react';
# leanpub-end-insert
import { ButtonInline } from './Button';
import './Story.css';

# leanpub-start-insert
const Story = ({ story, columns, archiveStore }) => {
# leanpub-end-insert
  const {
    title,
    url,
    author,
    num_comments,
    points,
    objectID,
  } = story;

  return (
    <div className="story">

      ...

      <span style={{ width: columns.archive.width }}>
# leanpub-start-insert
        <ButtonInline onClick={() => archiveStore.archiveStory(objectID)}>
# leanpub-end-insert
          Archive
        </ButtonInline>
      </span>
    </div>
  );
}

# leanpub-start-insert
export default inject('archiveStore')(observer(Story));
# leanpub-end-insert
~~~~~~~~

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
...

const Stories = ({ storyStore }) =>
  <div className="stories">
    <StoriesHeader columns={COLUMNS} />

    {(storyStore.readableStories || []).map(story =>
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

# leanpub-start-insert
export default inject('storyStore')(observer(Stories));
# leanpub-end-insert
~~~~~~~~

### Part 13: Local State with MobX

- first local state with React
- then refactrp to MobX

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

The `SearchStories` component will be a connected component. The next step is to implement that component. First, you start with a plain React component that has a form, input field and button.

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
      // TODO: fetch stories
      console.log(query);

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

- now lets use MobX for the local state

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import { observable, action } from 'mobx';
import { observer } from 'mobx-react';
# leanpub-end-insert
import Button from './Button';

# leanpub-start-insert
@observer
# leanpub-end-insert
class SearchStories extends Component {
# leanpub-start-insert
  @observable query = '';
# leanpub-end-insert

  constructor(props) {
    super(props);

    this.onChange = this.onChange.bind(this);
    this.onSubmit = this.onSubmit.bind(this);
  }

# leanpub-start-insert
  @action
# leanpub-end-insert
  onSubmit(event) {
# leanpub-start-insert
    if (this.query) {
# leanpub-end-insert
      // TODO: fetch stories
      console.log(this.query);
# leanpub-start-insert
      this.query = '';
# leanpub-end-insert
    }

    event.preventDefault();
  }

# leanpub-start-insert
  @action
# leanpub-end-insert
  onChange(event) {
    const { value } = event.target;
# leanpub-start-insert
    this.query = value;;
# leanpub-end-insert
  }

  render() {
    return (
      <form onSubmit={this.onSubmit}>
        <input
          type="text"
# leanpub-start-insert
          value={this.query}
# leanpub-end-insert
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

### Part 14: Interacting with an API

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import { observable, action } from 'mobx';
# leanpub-start-insert
import { observer, inject } from 'mobx-react';
# leanpub-end-insert
import Button from './Button';

# leanpub-start-insert
const HN_BASE_URL = 'http://hn.algolia.com/api/v1/search?query=';

const fetchStories = query =>
  fetch(HN_BASE_URL + query)
    .then(response => response.json());
# leanpub-end-insert

# leanpub-start-insert
@inject('storyStore') @observer
# leanpub-end-insert
class SearchStories extends Component {
  @observable query = '';

  constructor(props) {
    super(props);

    this.onChange = this.onChange.bind(this);
    this.onSubmit = this.onSubmit.bind(this);
  }

  @action
  onSubmit(event) {
# leanpub-start-insert
    const { storyStore } = this.props;
# leanpub-end-insert

    if (this.query) {
# leanpub-start-insert
      fetchStories(this.query)
        .then(result => storyStore.setStories(result.hits));
# leanpub-end-insert

      this.query = '';
    }

    event.preventDefault();
  }

  ...

}

export default SearchStories;
~~~~~~~~

{title="src/stores/storyStore.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { observable, computed, action } from 'mobx';
# leanpub-end-insert

const isNotArchived = archivedStoryIds => story =>
  archivedStoryIds.indexOf(story.objectID) === -1;

class StoryStore {
# leanpub-start-insert
  @observable stories = [];
# leanpub-end-insert

  constructor(rootStore) {
    this.rootStore = rootStore;
  }

# leanpub-start-insert
  @action setStories = stories => {
    this.stories = stories;
  }
# leanpub-end-insert

  @computed get readableStories() {
    const { archivedStoryIds } = this.rootStore.archiveStore;
    return this.stories.filter(isNotArchived(archivedStoryIds));
  }
}

export default StoryStore;
~~~~~~~~

### Part 15: Separation of API

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

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
import React, { Component } from 'react';
import { observable, action } from 'mobx';
import { observer, inject } from 'mobx-react';
import Button from './Button';
# leanpub-start-insert
import { fetchStories } from '../api/story';
# leanpub-end-insert

@inject('storyStore') @observer
class SearchStories extends Component {
  ...
}

...
~~~~~~~~

### Part 16: Error Handling

{title="src/components/SearchStories.js",lang="javascript"}
~~~~~~~~
...

@inject('storyStore') @observer
class SearchStories extends Component {

  ...

  @action
  onSubmit(event) {
    const { storyStore } = this.props;

    if (this.query) {
      fetchStories(this.query)
        .then(result => storyStore.setStories(result.hits))
# leanpub-start-insert
        .catch(error => storyStore.setError(error));
# leanpub-end-insert

      this.query = '';
    }

    event.preventDefault();
  }

  ...

}

...
~~~~~~~~

{title="src/stores/storyStore.js",lang="javascript"}
~~~~~~~~
import { observable, computed, action } from 'mobx';

const isNotArchived = archivedStoryIds => story =>
  archivedStoryIds.indexOf(story.objectID) === -1;

class StoryStore {
  @observable stories = [];
  @observable error = null;

  constructor(rootStore) {
    this.rootStore = rootStore;
  }

  @action setStories = stories => {
    this.stories = stories;
# leanpub-start-insert
    this.error = null;
# leanpub-end-insert
  }

# leanpub-start-insert
  @action setError = error => {
    this.stories = [];
    this.error = error;
  }
# leanpub-end-insert

  @computed get readableStories() {
    const { archivedStoryIds } = this.rootStore.archiveStore;
    return this.stories.filter(isNotArchived(archivedStoryIds));
  }
}

export default StoryStore;
~~~~~~~~

{title="src/components/Stories.js",lang="javascript"}
~~~~~~~~
...

const Stories = ({ storyStore }) =>
  <div className="stories">
    <StoriesHeader columns={COLUMNS} />

# leanpub-start-insert
    { storyStore.error && <p className="error">Something went wrong ...</p> }
# leanpub-end-insert

    {(storyStore.readableStories || []).map(story =>
      <Story
        key={story.objectID}
        story={story}
        columns={COLUMNS}
      />
    )}
  </div>

  ...
  ~~~~~~~~
