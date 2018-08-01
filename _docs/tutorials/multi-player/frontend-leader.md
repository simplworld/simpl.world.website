---
title: Build the Multi-player Game Leader UI
permalink: /docs/tutorials/multi-player/frontend-leader/
layout: docs
description:
---

## Build the Multi-player Game Leader UI

Often a leader is associated with more than one active run of a game. 
It will be helpful to show `Div` leaders the state of all their active runs as soon as they log in.

We will do this by presenting a table of the leader's active runs on the home page. If a run is in `Play` phase, an `Advance to Debrief` button will be available. 
If a run is in `Debrief` phase, the run's name will be linked to the run's debrief page.

Start by adding these actions to `js/actions/Actions.js`:

```jsx
// actions used to bracket processing a request (e.g. advancing the phase)
export const startProcessing = createAction('START_PROCESSING');
export const completeProcessing = createAction('COMPLETE_PROCESSING');

// advance run to next phase
export const advanceRunPhase = createAction('ADVANCE_RUN_PHASE', (run, ...args) => (
  AutobahnReact.call(`model:model.run.${run.id}.advance_phase`)
));

```
The `advanceRunPhase` action calls an RPC provided by the `simpl-modelservice` `Run` scope that advances the run to its next phase. 
The `startProcessing` and `completeProcessing` actions are used to prevent the user from resubmitting this request before it has completed.

Create a reducer `js/reducers/ProcessingReducer.js` to handle the `startProcessing` and `completeProcessing` actions:

```jsx
import {createReducer} from 'redux-create-reducer';
import recycleState from 'redux-recycle';

import {recyleStateAction} from 'simpl/lib/actions/state';

import {
  startProcessing,
  completeProcessing
} from '../actions/Actions';

const initial = {inProcess: false};

const processing = recycleState(createReducer(initial, {
  [startProcessing](state, action) {
    return Object.assign({}, state, {
      inProcess: true
    });
  },
  [completeProcessing](state, action) {
    // console.log("setting inProcess: false");
    return Object.assign({}, state, {
      inProcess: false
    });
  },
}), `${recyleStateAction}`);

export default processing

```

Then add `processing` to `reducers/combined/appReducers.js`:

```jsx
import {simplReducers} from 'simpl/lib/reducers/combined';
import {reducer as form} from 'redux-form';

import processing from '../ProcessingReducer';
import status from '../StatusReducer';

const reducers = simplReducers({
  form,
  // Add your customer reducers here, if any.
  processing,
  status
});

export default reducers;
```

Create a presentational component `js/components/AdvancePhase.js` to display the `Advance to Debrief` button:

```jsx
import React from 'react';
import {Button} from 'react-bootstrap';

import PropTypes from 'prop-types';

class AdvancePhase extends React.Component {
  render() {
    const phases = this.props.phases || [];

    const currentPhase = phases.find(
      (phase) => phase.id == this.props.run.phase
    );

    const nextPhase = phases.find(
      (phase) => phase.order == currentPhase.order + 1
    )

    if (nextPhase === undefined) {
      return (<span>{currentPhase.name}</span>);
    } else {
      if (this.props.advancingPhase) {
        return (
          <Button type='button'
             className="btn btn-sm btn-labeled btn-primary disabled">
            Advance to {nextPhase.name}
          </Button>
        );
      } else {
        return (
          <Button type='button'
             className="btn btn-sm btn-labeled btn-primary"
             onClick={() => this.props.advanceRunPhase(this.props.run)}>
            Advance to {nextPhase.name}
          </Button>
        );
      }
    }
  }
}

AdvancePhase.propTypes = {
  run: PropTypes.object.isRequired,
  phases: PropTypes.array,
  advanceRunPhase: PropTypes.func.isRequired,
  advancingPhase: PropTypes.bool.isRequired,
};

export default AdvancePhase;

```

and wrap it in a container component `js/containers/AdvancePhaseContainer`:

```jsx
import {connect} from 'react-redux';

import AdvancePhase from '../components/AdvancePhase';
import {
  advanceRunPhase,
  startProcessing,
  completeProcessing
} from '../actions/Actions';

function mapStateToProps(state) {
  return {
    phases: state.simpl.phase,
    advancingPhase: state.processing.inProcess
  };
}

function mapDispatchToProps(dispatch) {
  return {
    advanceRunPhase: function (run) {
      dispatch(startProcessing());   // disable all advance phase buttons
      dispatch(advanceRunPhase(run))
        .then((result) => {
          dispatch(completeProcessing());    // enable all advance phase  buttons
        });
    }
  };
}

const AdvancePhaseContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(AdvancePhase);

export default AdvancePhaseContainer;

```

Add a presentational component `js/components/RunRow.js` for displaying a run in a table row:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

import {Link} from 'react-router';

import AdvancePhaseContainer from '../containers/AdvancePhaseContainer';

class RunRow extends React.Component {
  constructor(props) {
    super(props);
    this.getPhase = this.getPhase.bind(this);
  }

  getPhase() {
    return this.props.phases.find(
      (p) => p.id == this.props.run.phase
    );
  }

  render() {
    const phase = this.getPhase();
    const linkDestination = `/run/${this.props.run.id}/debrief`;

    let name = this.props.run.name;
    if ((phase.name === 'Debrief')) {
      name = (
        <Link to={linkDestination} id={`Link_${this.props.run.name}`}>{this.props.run.name}</Link>
      );
    }

    return (
      <tr>
        <td>{name}</td>
        <td>
          <AdvancePhaseContainer run={this.props.run}/>
        </td>
      </tr>
    );
  }
}

RunRow.propTypes = {
  run: PropTypes.object.isRequired,
  phases: PropTypes.array.isRequired,
};

export default RunRow;

```

and wrap it in a container component `js/containers/RunRowContainer`:

```jsx
import {connect} from 'react-redux';

import RunRow from '../components/RunRow';

function mapStateToProps(state, ownProps) {
  const run = ownProps.run;
  return {
    phases: state.simpl.phase,
  };
}

const RunRowContainer = connect(
  mapStateToProps,
  null
)(RunRow);

export default RunRowContainer;

```

Finally, replace the contents of `js/modules/LeaderHome.js` with:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

import {connect} from 'react-redux';

import RunRowContainer from '../containers/RunRowContainer'

class LeaderHome extends React.Component {

  render() {
    const name = this.props.runuser.first_name + ' ' + this.props.runuser.last_name;
    const runRows = this.props.runs.map(
      (r) => <RunRowContainer key={r.id} run={r}/>
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
              <th width="60%">Run</th>
              <th width="30%">&nbsp;</th>
            </tr>
            </thead>
            <tbody>
            {runRows}
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
  runs: PropTypes.array,
};

function mapStateToProps(state) {
  return {
    runuser: state.simpl.current_runuser,
    runs: state.simpl.run
  };
}

const module = connect(
  mapStateToProps,
  null
)(LeaderHome);

export default module;

```

Now when you login as `leader@div.edu` with password `div`, you will see information about your active run:


![](/assets/img/tutorials/multi-player/Leader_Runs1.png){: width="60%" }

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

![](/assets/img/tutorials/multi-player/Leader_Runs2.png){: width="60%" }



## TODO Add LeaderDebriefRun.js that displays the last decisions and results for each world in the run during debrief.

We want leaders to be able display each world's latest decisions and results during while debriefing the run.  We'll next add a

C
