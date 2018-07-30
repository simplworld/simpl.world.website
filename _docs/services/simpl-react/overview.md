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
* The `simpl` layer. Uses the `wamp` layer to provide automatic data binding ans synchronization with `Scope`s defined in the modelservice implementation, provided it uses `Scope`s from the `modelservice` Python package.

You will mostly work on the `simpl` layer, and build your app by writing Presentational Components and tying up data to Container Components by using decorators.

### Decorators

* `simpl/lib/decorators/pubsub/publishes`
* `simpl/lib/decorators/pubsub/subscribes`
* `simpl/lib/decorators/rpc/calls`
* `simpl/lib/decorators/simpl`

### Components

#### Presentational Components

Presentational components only provide the necessary markup to render the UI.

#### Container Component

Container Components provide data and functions (`props`) and pass them down to their Presentational component.

#### WAMP Components

Wamp Components are convenience components that wrap a Smart or presentational components to provide them wamp-relative functionality, such as listening or publishing to a topic or calling a remote procedure on the model service

* TopicPublisher
* TopicSubscriber
* RPCCaller

### Reducers

#### The `Simpl.reducers.combined.simplReducers` reducer

This reducer provides the `simpl` and `router` reducers necessary for connecting to the modelservice and keeping state updated.

```js
import {simplReducers} from 'simpl/lib/reducers/combined';


const reducers = simplReducers({});

export default reducers;
```

You can add your own reducers to it by passing them as arguments:

```js
import {simplReducers} from 'simpl/lib/reducers/combined';

import myreducer from '../MyReducers'

const reducers = simplReducers({
  myreducer,
});

export default reducers;
```

### Stores

#### The `Simpl.stores.finalCreateStoreFactory`

This factory returns a `createStore` function with all the necessary devTools configured according to the environment.

``stores/AppStore.js``

```js
import { createStore } from 'redux'

import {finalCreateStoreFactory} from 'simpl/lib/stores';

import rootReducer from '../reducers/combined/appReducers'

export default function configureStore(initialState) {

  const finalCreateStore = finalCreateStoreFactory(process.env.NODE_ENV)
  const store = finalCreateStore(rootReducer, initialState);

  return store;
};
```

`apps/main.js`

```js
import configureStore from '../stores/AppStore'


const store = configureStore();
```
