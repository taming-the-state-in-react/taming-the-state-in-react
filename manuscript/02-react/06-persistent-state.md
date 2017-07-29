{pagebreak}

## Persistence in State

State in applications is often not persistent. When your application starts, there is often an initial state. The state updates when the user interacts with the application or data arrives from a backend application. However, you might wonder whether there is a way to persist the state? The question applies to both, local state management and sophisticated state management later on.

The obvious answer to this question would be to implement a backend application with a database to persist the state. Extracting the state from your application is called **dehydrating state**. Now, every time your application bootstraps, you would retrieve the state from the backend that keeps it in a database. Once the state arrives asynchronously in the response, you would **rehydrate state** into your application.

While the dehydration of the state could happen any time your application is running, the rehydration would take place when your components mount. The best place to do it in React would be the `componentDidMount()` lifecycle method. Take for example the `ArchiveableList` component. It could retrieve all the already archived unique identifiers of items on `componentDidMount()` and rehydrate it to the local state.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class ArchiveableList extends React.Component {
  constructor(props) {
    ...
  }

  onArchive(id) {
    ...
  }

  componentDidMount() {
    fetch('path/to/archived/items')
      .then(response => response.json())
      .then(archivedItems => this.setState(rehydrateArchivedItems(archivedItems)));
  }

  render() {
    ...
  }
}

function rehydrateArchivedItems(archivedItems) {
  return function(prevState) {
    return {
      archivedItems: [
        ...prevState.archivedItems,
        ...archivedItems
      ]
    };
  };
}
~~~~~~~~

Now, every time the component initializes, the persistent archived items will get rehydrated into the application state.

The dehydration could happen anytime, but to avoid inconstencies, in the example of archived items, the dehydration would take place when an item gets archived. It is an usual request to the backend to save the item as being archived.

The rehydration and dehydration of state are most often unconcious steps in modern applications. It is common sense to retrieve all the necessary data from the backend when your application bootstraps and to update the data when something has changed. But you can keep the rehydration and dehydration of state in mind to keep your application state in sync with your backend data as single source of truth.

### Local Storage

Is there a more lightweight solution compared to a backend application? You could use the native browser API. To be more specific, most of the modern browser have a storage functionality to persist data. It is the lightwight version of a database that is used in the browser. Of course, it is only visible to the user of the browser and cannot be distributed to other users.

Modern browsers have access to the [local storage](https://developer.mozilla.org/en/docs/Web/API/Window/localStorage) and [session storage](https://developer.mozilla.org/en/docs/Web/API/Window/sessionStorage). Both work the same, but there is one difference in their functionalities. While the local storage keeps the data even when the browser is closed, the session storage expires once the browser closes. Both storages work the same by using key value pairs.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// Save data to localStorage
localStorage.setItem('key', 'value');

// Get saved data from localStorage
var data = localStorage.getItem('key');

// Remove saved data from localStorage
localStorage.removeItem('key');

// Remove all saved data from localStorage
localStorage.clear();
~~~~~~~~

In the end, you can apply them the same way as you did in the previous `ArchiveableList` component that used the backend request to retrieve the data. Only that the `ArchiveableList` component would use the storage instead of the backend to retrieve the state. If you are keen to explore the usage with the local storage in React, you can read more about it in [this article](https://www.robinwieruch.de/local-storage-react/).

### Caching in State

The local state, later on the sophisticated state as well, can be used as a cache for your application. A cache would make recurring requests to retrieve data from a backend redundant, because they would return the same data as before and the data is already cached in the state.

Imagine your application has an interface to search for popular stories on a news platform. The news platform has an open API that you can use to retrieve those popular stories. Your own application that consumes the API of this platform would only have a search field to search for popular stories from the platform and a list to display the stories once you have searched for them.

Next imagine that you make you first request searching popular stories about "React". You are not satisfied with the search result, because you wanted to be more specific, and search again for "React Local State". Still, no satisfying search result. Now the search result for "React Local State" is visible in your application. Next you want to head back to search for "React" stories again. Your application makes a third request to retrieve the "React" stories from the platform API. In a perfect world, the application would know that you have already searched for "React" stories before. That's where caching comes into play. The third request could be avoided when the application would have cached the search results.

Such a fluctuant cache solution is not difficult to implement with a local state. Bear in mind that it would work with a sophisticated state management layer that is explained later in the book too. When searching for the stories, you already have an unique identifier which you can use as a key in an object to store the search result in the local state. The unique identifier is your search term. It would be either "React" or "React Local State" considering the previous example. The value corresponending to the key would be the search result. In the example, it would be the popular stories about "React" and "React Local Storage". After all, your cache object in the local state might be similar to this:

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.state = {
  ...
  searchCache: {
    "React": [...],
    "ReactLocalState": [...],
  }
}
~~~~~~~~

Every time your application performs a search request, the key value pair in the cache object in your local state would be filled. Before you make a new request, the cache would be checked if the search term is already available as a key. If the key is available, the request would be surpressed and the cache result would be used instead. If the key is not available, a request would be made. After the request succeeded, the search term would be saved as key and the search result would be saved as value for the key in the local state.

The book doesn't give you an in-depth implementation of the cache solution. If you did read [the Road to learn React](https://www.robinwieruch.de/the-road-to-learn-react/), you will already know how to implement such a cache in plain React with local state. In one of its lessons, the book uses a cache in a more elaborated way to cache paginated search results efficiently in the local state.