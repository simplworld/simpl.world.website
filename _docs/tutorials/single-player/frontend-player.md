---
title: Build the Single-player Game Player UI
permalink: /docs/tutorials/single-player/frontend-player/
layout: docs
description:
---

## Build the Single-player Game Player UI

Players need to be able to submit decisions and see their results.

Create an action in `js/actions/Actions.js` for publishing decisions to the `submit_decision` topic defined by calc-model `game/games.py`:

```jsx
import {createAction} from 'redux-actions';

import AutobahnReact from 'simpl-react/lib/autobahn';

// submit player decision and advance to next period
export const submitDecision =
  createAction('SUBMIT_DECISION', (period, operand, ...args) =>
    AutobahnReact.call(`model:model.period.${period.id}.submit_decision`, [operand])
  );

```
**NOTE**: The action publishes to this topic because the calc-model `game/games.py` `submit_decision` endpoint subscribes to the topic.

Create a presentational component `js/components/DecisionForm.js` for entering player decisions:

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
              required={true}
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

Wrap it in a container component `js/containers/DecisionFormContainer.js`:

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
        .then((result) => {
          const status = result.payload;
          if (status !== 'ok') {
            console.log("DecisionFormContainer.submitDecision failed due to: ", status);
          } else {
            console.log("DecisionFormContainer.submitDecision succeeded");
          }
        });
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

Now, when a player logs in, they see a form for entering decisions and a logout link:

![](/assets/img/tutorials/single-player/Player_Home.png)

As the player submits decisions, the redux state automatically updates with new periods, decisions, and results:

![](/assets/img/tutorials/single-player/Simpl_Player_Home.png){: width="100%" }

Congratulations! You are now ready to build the [Single-player Game Leader UI]({% link _docs/tutorials/single-player/frontend-leader.md %})



