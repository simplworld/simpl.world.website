---
title: Build the Player Multi-player Game UI
permalink: /docs/tutorials/multi-player/frontend-player/
layout: docs
description:
---

## Build the Player Multi-player Game UI


First, create actions in `js/actions/Actions.js` for submitting decisions to the `submit_decision` topic defined by div-model `game/games.py` 
and for setting/clearing a status message it returns.

```jsx
import {createAction} from 'redux-actions';

import AutobahnReact from 'simpl/lib/autobahn';

// actions for setting / clearing a status message
export const setStatus = createAction('SET_STATUS');
export const clearStatus = createAction('CLEAR_STATUS');

// submit player decision then calculate result if both dividend and divisor have been submitted
export const submitDecision =
    createAction('SUBMIT_DECISION', (period, operand, ...args) =>
        AutobahnReact.call(`model:model.period.${period.id}.submit_decision`, [operand])
    );

```
**NOTE** the `submit_decision`action calls this topic because the div-model `game/games.py` `submit_decision` endpoint registers as an RPC on the topic. 
Because `submit_decision` validates the operand, calling an RPC allows us to check the returned status for an error.

Create a presentation component `js/components/DecisionForm.js` for entering player decisions and displaying an error message:

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
            >Submit Decision</Button>
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

wrap it in a container component that checks the returned status `js/containers/DecisionFormContainer.js`:

```jsx
import {connect} from 'react-redux';
import {withRouter} from 'react-router';

import DecisionForm from '../components/DecisionForm';

import {submitDecision, setStatus} from '../actions/Actions';

function mapStateToProps(state, ownProps) {
  const initialValues = {
    'operand': ownProps.operand
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
      dispatch(submitDecision(ownProps.period, operand))
        .then((result) => {
          const status = result.payload;
          if (status !== 'ok') {
            console.log("DecisionFormContainer.submitDecision failed due to: ", status);
            dispatch(setStatus(status));
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

In your js/modules/PlayerHome.js, replace the original contents with:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

import {connect} from 'react-redux';

import DecisionFormContainer from '../containers/DecisionFormContainer'

// import StatusNotificationContainer from '../containers/StatusNotificationContainer';

class PlayerHome extends React.Component {
  render() {
    const quotient = (this.props.result) ? this.props.result.data.quotient : '';
    const other_operand = (this.props.other_decision) ? this.props.other_decision.data.operand : '';
    const operand = (this.props.decision) ? this.props.decision.data.operand : 0;
    let play = '';
    if (this.props.canPlay) {
      play = (
        <div>
          <br/>
          <p>You are in charge of submitting a valid {this.props.runuser.role_name}.</p>
          < DecisionFormContainer period={this.props.period} operand={operand}/>
          {/*<StatusNotificationContainer/>*/}
        </div>
      );
    } else {
      play = (
        <div>
          <span>World {this.props.runuser.role_name}: {operand}</span>
        </div>
      );
    }

    return (
      <div>
        <h1>Hello Player: {this.props.runuser.email}</h1>
        <span>World Quotient: {quotient} </span><br/>
        <span>World {this.props.other_role_name}: {other_operand}</span><br/>
        {play}
        <br/>
        <a href="/logout/" className="btn btn-success btn-lg">Logout</a>
      </div>
    );
  }
}

PlayerHome.propTypes = {
  canPlay: PropTypes.bool.isRequired,
  runuser: PropTypes.object.isRequired,
  other_role_name: PropTypes.string.isRequired,
  period: PropTypes.object.isRequired,
  decision: PropTypes.object,
  other_decision: PropTypes.object,
  result: PropTypes.object,
};

function mapStateToProps(state) {
  const runuser = state.simpl.current_runuser;

  const run = state.simpl.run.find(
    (r) => r.id === runuser.run
  )
  const currentPhase = state.simpl.phase.find(
    (p) => p.id === run.phase
  )
  const playPhase = state.simpl.phase.find(
    (p) => p.name === 'Play'
  )
  const canPlay = playPhase.id === currentPhase.id;
  console.log("PlayerHome: currentPhase=", currentPhase, ", playPhase.id=", playPhase.id, ", canPlay=", canPlay);

  const other_role_name = runuser.other_roles[0];

  const scenario = state.simpl.scenario.find(
    (s) => runuser.world === s.world
  );

  const period = state.simpl.period.find(
    (p) => scenario.id === p.scenario
  );

  const decision = state.simpl.decision.find(
    (d) => period.id === d.period && d.role === runuser.role
  );

  const other_decision = state.simpl.decision.find(
    (d) => period.id === d.period && d.role !== runuser.role
  );

  const result = state.simpl.result.find(
    (r) => period.id === r.period
  );

  return {
    canPlay,
    runuser,
    other_role_name,
    period,
    decision,
    other_decision,
    result
  };
}

const module = connect(
  mapStateToProps,
  null
)(PlayerHome);

export default module;
```

Now when a player logs in, they see a form for entering decisions for their role and a logout link:

![](/assets/img/tutorials/multi-player/Players_Home.png)

We'll implement the commented out `StatusNotificationContainer` at a later time.

As a world's players submit decisions, the redux state automatically updates with new periods, decisions and results:

<img src="/assets/img/tutorials/multi-player/Simpl_Players_Home.png" width="100%">


Congratulations! You are now ready to [Build the Leader Multi-player Game UI]({% link _docs/tutorials/multi-player/frontend-leader.md %})