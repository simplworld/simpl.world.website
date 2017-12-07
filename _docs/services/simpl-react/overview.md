---
title: Simpl-React Overview
permalink: /docs/services/simpl-react/
layout: docs
description:
---

## Overview

## Abstraction Layers

This module is conceptually organized in two layers:

* The `wamp` layer. Includes primitives to publish/subcribe and remote call via the wamp protocol.
* The `simpl` layer. Uses the `wamp` layer to provide automatic data binding ans synchronization with `Scope`s defined in the modelservice implementation, provided it uses `Scope`s from the `modelservice` Python package.

You will mostly work on the `simpl` layer, and build your app by writing Presentational Components and tying up data to Smart Components by using decorators.

## Decorators

* `simpl/lib/decorators/pubsub/publishes`
* `simpl/lib/decorators/pubsub/subscribes`
* `simpl/lib/decorators/rpc/calls`
* `simpl/lib/decorators/simpl`

## Components

### Presentational Components

Presentational components only provide the necessary markup to render the UI.

### Smart Component

Smart Components provide data and functions (`props`) and pass them down to their Presentational component.

### Field Components

`Simpl` includes some convenience components from building forms, such as:

* `EmailInput`
* `IntegerInput`
* `FloatInput`

For more information, see the [Forms docs](./forms/overview.md).

### WAMP Components

Wamp Components are convenience components that wrap a Smart or presentational components to provide them wamp-relative functionality, such as listening or publishing to a topic or calling a remote procedure on the model service

* TopicPublisher
* TopicSubscriber
* RPCCaller

## Reducers

### The `Simpl.reducers.combined.simplReducers` reducer

This reducer provides the `simpl` and `router` reducers necessary for connecting to the modelservice and keeping state updated.

```javascript
import {simplReducers} from 'simpl/lib/reducers/combined';


const reducers = simplReducers({});

export default reducers;
```

You can add your own reducers to it by passing them as arguments:

```javascript
import {simplReducers} from 'simpl/lib/reducers/combined';

import myreducer from '../MyReducers'

const reducers = simplReducers({
  myreducer,
});

export default reducers;
```

## Stores

### The `Simpl.stores.finalCreateStoreFactory`

This factory returns a `createStore` function with all the necessary devTools configured according to the environment.

``stores/AppStore.js``

```javascript
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

```javascript
import configureStore from '../stores/AppStore'


const store = configureStore();
```
