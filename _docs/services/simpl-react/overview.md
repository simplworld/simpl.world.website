---
title: Simpl-React Overview
permalink: /docs/services/simpl-react/
layout: docs
description:
---

## Simpl-React Overview

### Abstraction Layers

This module is conceptually organized in two layers:

* The `wamp` layer. Includes primitives to publish/subcribe and remote call via the wamp protocol.
* The `simpl` layer. Uses the `wamp` layer to provide automatic data binding ansd synchronization with 
`Scope`s defined in the modelservice implementation, provided it uses `Scope`s from the `modelservice` Python package.

You will mostly work at the `simpl` layer using the `simpl` decorator to wrap your root component. You then build 
your [single-page application](https://en.wikipedia.org/wiki/Single-page_application) by writing Presentational Components 
tied to data provided by Container Components

### Simpl Decorators

* `simpl/lib/decorators/simpl`
* `simpl/lib/decorators/pubsub/publishes`
* `simpl/lib/decorators/pubsub/subscribes`
* `simpl/lib/decorators/rpc/calls`

### Components

#### Presentational Components

Presentational components only provide the necessary markup to render the UI.

#### Container Components

Container Components provide data and functions (`props`) and pass them down to their Presentational component.

#### WAMP Components

Wamp Components are convenience components that wrap container or presentational components to provide them 
wamp-relative functionality, such as listening or publishing to a topic or calling a remote procedure on the model service

* TopicPublisher
* TopicSubscriber
* RPCCaller

### Redux Reducers

The `Simpl.reducers.combined.simplReducers` reducer provides the `simpl` and `routing` reducers necessary 
for connecting to the modelservice and keeping state updated.

The `cookiecutter-ui-template` cookiecutter template uses `simplReducers` in  `js/reducers/combined/appReducers.js`:
```jsx
import {simplReducers} from 'simpl/lib/reducers/combined';
import {reducer as form} from 'redux-form';

const reducers = simplReducers({
  form
  // Add your customer reducers here, if any.
});

export default reducers;

```

You can configure your own reducers by passing them as arguments:

```jsx
import {simplReducers} from 'simpl/lib/reducers/combined';

import myreducer from '../MyReducers'

const reducers = simplReducers({
  myreducer
});

export default reducers;
```

### Redux Store

The `Simpl.stores.finalCreateStoreFactory` returns a `createStore` function with all the necessary devTools 
configured according to the environment.

The `cookiecutter-ui-template` cookiecutter template uses `finalCreateStoreFactory` in `js/stores/appStore.js`:

```jsx
/* eslint global-require: "off" */
import {finalCreateStoreFactory} from 'simpl/lib/stores';

import rootReducer from '../reducers/combined/appReducers';


export default function configureStore(initialState, node_env) {
  const finalCreateStore = finalCreateStoreFactory(node_env || process.env.NODE_ENV);
  const store = finalCreateStore(rootReducer, initialState);

  if (module.hot) {
    // Enable Webpack hot module replacement for reducers
    module.hot.accept('../reducers/combined/appReducers', () => {
      const nextReducer = require('../reducers/combined/appReducers');
      store.replaceReducer(nextReducer);
    });
  }

  return store;
}
```

The `cookiecutter-ui-template` cookiecutter template also provides `js/main.js` that configures the application's store and 
defines the root of your single-page application:

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import {Provider} from 'react-redux';

import configureStore from './stores/AppStore';
import Root from './modules/Root';

const store = configureStore();

ReactDOM.render(
  <Provider store={store}>
    <Root />
  </Provider>,
  document.getElementById('App')
);


```


