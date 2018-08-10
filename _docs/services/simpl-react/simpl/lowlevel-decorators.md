---
title: "Simpl-React: Using the Simpl Decorator<"
permalink: /docs/services/simpl-react/simpl/low-level-decorators/
layout: docs
description:
---

## Using the Simpl Decorator


The `simpl` decorator allows your app to stay in sync with relevant changes by subscribing to notifications from the modelservice. 
The UI will receive an action and its store will be updated every time a Scope within a subscribed topic is added, updated or deleted.

In addition to [defining your combined Redux reducers and store]({% link _docs/services/simpl-react/overview.md %}), 
you will need a Presentational component that displays progress in loading Simpl data into the Redux store. 
This component depends on the simpl-react `CONNECTION_STATUS` property to track this progress.
The `simpl-ui-cookiecutter` cookiecutter template provides such a progress component in `js/components/Progress.js`.

To connect your application to its Redux store and reducers, you must wrap your root Container component in the `simpl` decorator. 
The `simpl-ui-cookiecutter` cookiecutter template does this for you by providing `js/modules/Root.js`:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

import {connect} from 'react-redux';
import {Router, Route, browserHistory} from 'react-router';

import {SimplActions} from 'simpl/lib/actions';
import {simpl} from 'simpl/lib/decorators/simpl';

import Progress from '../components/Progress';

...

const RootContainer = connect(
  null,
  null
)(Root);

const runs = RUNS.map((id) => `model:model.run.${id}`);
const runusers = RUNUSERS.map((id) => `model:model.runuser.${id}`);
const worlds = WORLDS.map((id) => `model:model.world.${id}`);
const topics = (LEADER) ? runs : runusers.concat(worlds);
console.log("topics: ", topics);

export default simpl({
  authid: AUTHID,
  password: 'nopassword',
  url: `${MODEL_SERVICE}`,
  progressComponent: Progress,
  root_topic: ROOT_TOPIC,
  topics: () => topics,
  loadAllScenarios: false
})(RootContainer);

```

The `topics` argument passed to the `simpl` decorator is initialized from Javascript constants defined in the `frontend` Django app's 
`templates/frontend/home.html` using data returned by the `frontend` app's `HomeView.get_context_data` method in `views.py`'`.

The `loadAllScenarios` argument passed to the `simpl` decorator indicates whether the current user should have access to 
other users' scenarios. The default setting of `false` makes sense for multi-player games. Players in multi-player games 
typically have access to their own scenarios and their world's shared scenarios. In multi-player games, leaders typically 
have access to their own scenarios and to all world scenarios. 
