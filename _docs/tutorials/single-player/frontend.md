---
title: Build the Single-player Game Frontend UI
permalink: /docs/tutorials/single-player/frontend/
layout: docs
description:
---

## Build the Single-player Game Frontend

### Prerequisites

This tutorial assumes you are familiar with [react](https://reactjs.org) and [redux](https://github.com/reduxjs/react-redux).

You will need to have these installed:

* [Node](https://nodejs.org) >= 5.7.0
* [NPM](https://nodejs.org) >= 3.6.0
* Python >= 3.6
* [virtualenv](https://virtualenv.pypa.io/en/stable/)

Have the [Games API service]({% link _docs/getting-started.md %}) running on http://localhost:8100/ and 
the [calc-model service]({% link _docs/tutorials/single-player/modelservice.md %}) running on http://localhost:8080. 

### Installation

In a separate terminal, create a new virtualenv called 'calc-ui':

```shell
$ mkvirtualenv calc-ui
```

Install the `cookiecutter` Python package:

```shell
$ pip install cookiecutter
```

Use `cookiecutter` to create the boilerplate for your game Frontend.

```shell
$ cookiecutter https://github.com:simplworld/simpl-ui-cookiecutter.git
```

For the `game_slug` value, enter `calc` (the slug value you used in the [modelservice tutorial]({% link _docs/tutorials/single-player/modelservice.md %})). 
For all other values, you can use the default or choose your own.

For example,

```shell
project_name [Simulation UI]: Calc UI
repo_slug [calc-ui]:
project_slug [calc_ui]:
game_slug [calc-ui]: calc
modelservice_slug [calc-model]:
topic_root [world.simpl]:
app_slug [frontend]:
version [0.1.0]:
```

After the project layout is created, install the requirements:

```shell
$ cd calc-ui
$ pip install -r requirements.txt
```

In a separate terminal, install the project's JavaScript node modules and run gulp to keep the web server's Javascript updated as you work on the frontend:

```shell
$ npm install
$ gulp
```

To aid development, you can install the following Chrome DevTools Extensions:

* [Redux DevTools Extension](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)
* [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)

It is also recommended you configure your editor to integrate with ESLint:

* [PyCharm](https://www.jetbrains.com/help/pycharm/2016.1/eslint.html)
* [SublimeText2](https://github.com/roadhump/SublimeLinter-eslint)

### Configuration

Like most websites, the frontend service will need a place where it can store information about sessions and their users. 
The user's specific information will be fetched from the [Simple Games API]({% link _docs/getting-started.md %}) and kept in sync automatically.

**NOTE** For the purposes of this tutorial we're going to use SQLite, but this can be changed to match whatever database backend you prefer. 
In a production environment, you'll likely want to switch to something like PostgreSQL.

First, let's create the necessary local tables:

```shell
$ ./manage.py migrate
```

Then, start your frontend service with:

```shell
$ ./manage.py runserver 0.0.0.0:8000
```

In Chrome, head to `http://localhost:8000/` and login as `s1@calc.edu` with password `s1`.
Once you are logged in, you should see the 'Hello Player' message of the skeleton app.

![](/assets/img/tutorials/single-player/Hello_Player.png)

Open Chrome's DevTools, and select the 'Redux' tab. You will see a list of actions, and the
current `state` of the store. Note that the state has a property named `simpl`. Expand the `simpl` property
to see all the scope properties associated with the current user.

<img src="/assets/img/tutorials/single-player/Hello_Simpl_Player.png" width="100%">

These properties will be updated as the model service adds, removes or updates scopes.
You will connect your components to the properties and they will update accordingly.

Next, logout by going to `localhost:8000/logout/` in your browser. Then login as `leader@calc.edu` with password `leader`.
Once you are logged in, you should see the 'Hello Leader' message of the skeleton app. If you look at the `simpl`
state properties, you'll see all the run's runusers have been loaded. No player scenarios have been loaded.

<img src="/assets/img/tutorials/single-player/Hello_Simpl_Leader1.png" width="100%">

In a multi-player simulation, players are assigned to a world with other players.
The cookiecutter template assumes you are implementing a multi-player simulation in which players
see only their own and their world's data. By default, leaders can see the worlds and users in their subscribed runs,
but not the scenarios of other users. Consequently, the template `js/modules/Root.js` sets the simpl decorator's 
`loadAllScenarios` argument false to block access to scenarios of other users.

We want Calc leaders to have access to all information on all players in their runs. Since Calc players are not associated with
a world, we need to modify the template `js/modules/Root.js` code to load all player scenarios for leaders.

In your `js/modules/Root.js`:

```jsx
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

change the simpl decorator's `loadAllScenarios` argument to LEADER:

```jsx
export default simpl({
  authid: AUTHID,
  password: 'nopassword',
  url: `${MODEL_SERVICE}`,
  progressComponent: Progress,
  root_topic: ROOT_TOPIC,
  topics: () => topics,
  loadAllScenarios: LEADER
})(RootContainer);
```

Refresh your Chrome browser page and you'll see all the run's player scenarios have been loaded into the `simpl` state.

<img src="/assets/img/tutorials/single-player/Hello_Simpl_Leader2.png" width="100%">

### Implementation

To implement your UI, you will write [Container Components and Presentational Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0).

The Presentational Components will provide the necessary markup to render UI elements, while the Container Components will wrap them providing the necessary data.

First, create an action in `js/actions/Actions.js` for submitting decisions to the `submit_decision` topic defined by calc-model `game/games.py`.

```jsx
import {createAction} from 'redux-actions';

import AutobahnReact from 'simpl/lib/autobahn';

// submit player decision and advance to next period
export const submitDecision =
    createAction('SUBMIT_DECISION', (period, operand, ...args) =>
        AutobahnReact.publish(`model:model.period.${period.id}.submit_decision`, [operand])
    );

```
Note the action publishes to this topic because the calc-model `game/games.py `submit_decision` endpoint subscribes to the topic.

Create a presentation component `js/components/DecisionForm.js` for entering player decisions:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

import {Button} from 'react-bootstrap';

import {Field, reduxForm} from 'redux-form';

class DecisionForm extends React.Component {

  constructor(props) {
    super(props);
    this.submitForm = this.submitForm.bind(this);
  }

  submitForm(values) {
    this.props.submitDecision(values);
  }

  render() {
    const {error, handleSubmit, submitting, invalid} = this.props;
    return (
      <div className="content-wrapper">
        <form id="testScenarioForm" onSubmit={handleSubmit(this.submitForm)}>

          <div className="form-group">
            <label>Enter a number:</label>
            <Field
              component="input"
              type="number"
              name="operand"
              id="operand"
              required="true"
              step="any"
            />
          </div>

          <div>
            <Button
              type="submit"
              bsClass="btn btn-mr btn-labeled btn-success"
              bsStyle="success"
              disabled={invalid || submitting}
            >Add to Total</Button>
          </div>

          <div>
            {error && <div><p>{error}</p></div>}
          </div>

        </form>
      </div >
    );
  }
}

DecisionForm.propTypes = {
  runuser: PropTypes.object.isRequired,
  initialValues: PropTypes.object.isRequired,

  // redux-form props
  handleSubmit: PropTypes.func,
  invalid: PropTypes.bool,
  submitting: PropTypes.bool,
  error: PropTypes.string,

  // dispatch actions
  submitDecision: PropTypes.func.isRequired
};

export default reduxForm({
  form: 'decisionForm'
})(DecisionForm);

```

wrap it in a container component `js/containers/DecisionFormContainer.js`

```jsx
import {connect} from 'react-redux';
import {withRouter} from 'react-router';

import DecisionForm from '../components/DecisionForm';

import {submitDecision} from '../actions/Actions';

function mapStateToProps(state, ownProps) {
  const initialValues = {
    'operand': 0
  };
  return {
    runuser: state.simpl.current_runuser,
    initialValues
  };
}

function mapDispatchToProps(dispatch, ownProps) {
  return {
    submitDecision(values) {
      // submit player's decision
      const operand = values.operand;
      dispatch(submitDecision(ownProps.currentPeriod, operand))
    }
  };
}

const DecisionFormContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(DecisionForm);

export default withRouter(DecisionFormContainer);
```

In your `js/modules/PlayerHome.js`, replace the original contents with:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

import {connect} from 'react-redux';

import DecisionFormContainer from '../containers/DecisionFormContainer'

class PlayerHome extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello Player: {this.props.runuser.email}</h1>
        <p>Current total: {this.props.total}</p>
        <DecisionFormContainer currentPeriod={this.props.currentPeriod}/>
        <br/>
        <a href="/logout/" className="btn btn-success btn-lg">Logout</a>
      </div>
    );
  }
}

PlayerHome.propTypes = {
  runuser: PropTypes.object.isRequired,
  total: PropTypes.number,
  currentPeriod: PropTypes.object.isRequired,
};

function mapStateToProps(state) {
  const runuser = state.simpl.current_runuser;

  const scenario = state.simpl.scenario.find(
    (s) => runuser.id === s.runuser
  );

  const unsortedPeriods = state.simpl.period.filter(
    (p) => scenario.id === p.scenario
  );
  const periods = _.sortBy(unsortedPeriods, (p) => p.order);
  const periodOrder = _.last(periods).order;

  let total = 0;
  if (periodOrder > 1) { // pull total from last result
    const lastPeriod = periods[periodOrder - 2];
    const lastResult = state.simpl.result.find(
      (s) => lastPeriod.id === s.period
    );
    total = lastResult.data.total;
  }

  const currentPeriod = periods[periodOrder - 1];

  return {
    runuser,
    total,
    currentPeriod
  };
}

const module = connect(
  mapStateToProps,
  null
)(PlayerHome);

export default module;
```

Now when a player logs in, they see a form for entering decisions and a logout link:

![](/assets/img/tutorials/single-player/Player_Home.png)

As the player submits decisions, the redux state automatically updates with new periods, decisions and results:

<img src="/assets/img/tutorials/single-player/Simpl_Player_Home.png" width="100%">

We want leaders to be able see the player results. We'll next update the leader home page so they can.

Create a presentation component `js/components/PlayerResultRow.js` for displaying one player's results:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

function PlayerResultRow(props) {
  return (
    <tr>
      <td>{props.runuser.email}</td>
      <td>{props.periodsPlayed}</td>
      <td>{props.total}</td>
    </tr>
  );
}

PlayerResultRow.propTypes = {
  runuser: PropTypes.object.isRequired,
  periodsPlayed: PropTypes.number.isRequired,
  total: PropTypes.number.isRequired
};

export default PlayerResultRow;
```

wrap it in a container component `js/containers/PlayerResultRowContainer.js`

```jsx
import {connect} from 'react-redux';
import {withRouter} from 'react-router';

import PlayerResultRow from '../components/PlayerResultRow';

function mapStateToProps(state, ownProps) {
  const runuser = ownProps.runuser;

  const scenario = state.simpl.scenario.find(
    (s) => runuser.id === s.runuser
  );

  let periodsPlayed = 0;
  let total = 0;
  if (scenario) { // avoid runtime errors while state is loading
    const unsortedPeriods = state.simpl.period.filter(
      (p) => scenario.id === p.scenario
    );
    const periods = _.sortBy(unsortedPeriods, (p) => p.order);
    const periodOrder = _.last(periods).order;

    if (periodOrder > 1) { // pull total from last result
      const lastPeriod = periods[periodOrder - 2];
      periodsPlayed = lastPeriod.order;

      const lastResult = state.simpl.result.find(
        (s) => lastPeriod.id === s.period
      );
      total = lastResult.data.total;
    }
  }

  return {
    runuser,
    periodsPlayed,
    total
  };
}

const PlayerResultRowContainer = connect(
  mapStateToProps,
  null
)(PlayerResultRow);

export default withRouter(PlayerResultRowContainer);
```

In your `js/modules/LeaderHome.js`, replace the original contents with:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

import {connect} from 'react-redux';

import PlayerResultRowContainer from '../containers/PlayerResultRowContainer'

class LeaderHome extends React.Component {

  render() {
    const name = this.props.runuser.first_name + ' ' + this.props.runuser.last_name;
    const playerRows = this.props.players.map(
      (p) => <PlayerResultRowContainer key={p.id} runuser={p}/>
    );
    return (
      <div>
        <div>
          <h1>Hello {name}</h1>
        </div>
        <div>
          <table>
            <thead>
            <tr>
              <th width="30%"> Player</th>
              <th width="30%"> Periods Played</th>
              <th width="30%"> Total</th>
            </tr>
            </thead>
            <tbody>
            {playerRows}
            </tbody>
          </table>
        </div>
        <br/>
        <a href="/logout/" className="btn btn-success btn-lg">Logout</a>
      </div>
    );
  }
}

LeaderHome.propTypes = {
  runuser: PropTypes.object.isRequired,
  players: PropTypes.array.isRequired
};

function mapStateToProps(state) {
  const runuser = state.simpl.current_runuser;

  const unsortedPlayers = state.simpl.runuser.filter(
    (ru) => runuser.id !== ru.id
  );
  const players = _.sortBy(unsortedPlayers, (p) => p.email);

  return {
    runuser,
    players
  };
}

const module = connect(
  mapStateToProps,
  null
)(LeaderHome);

export default module;
```

Now when a leader logs in, they see the current player results:

![](/assets/img/tutorials/single-player/Leader_Home.png)

Let's add some styling to make it easier to read the table of results.

In `frontend/templates/frontend.home.html`, replace

```html
  <head>
  </head>
```

with

```html
  <head>
    <style>
    table, th, td { border: 1px solid black;  }
    </style>
  </head>
```

![](/assets/img/tutorials/single-player/Leader_Home2.png)

To see the revised leader page in action, open an incognito window and login into http://localhost:8000/ as `s2@calc.edu` with password `s2`.

<img src="/assets/img/tutorials/single-player/Simpl_Play.png" width="100%">

Submit a decision in the `s2@calc.edu` window. The simpl state in both browser windows will update with a new result causing the leader home page to update accordingly.

<img src="/assets/img/tutorials/single-player/Simpl_Play2.png" width="100%">

This concludes our tutorial! We have barely scratched the surface. A completed example implementation is available at 
[https://github.com/simplworld/simpl-calc-ui]<!--(https://github.com/simplworld/simpl-calc-ui)-->
that uses the game slug `simpl-calc`.

There's much more that you can do with Simpl. For more informations, check out the service-specific documentation.
