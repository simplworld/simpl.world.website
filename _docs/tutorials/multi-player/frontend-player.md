---
title: Build the Multi-player Game Player UI
permalink: /docs/tutorials/multi-player/frontend-player/
layout: docs
description:
---

## Build the Multi-player Game Player UI

Players need to be able to submit decisions for their assigned role and see their world's results. They also need to be notified if a value they submitted was not valid.

First, create three actions in `js/actions/Actions.js`. One for submitting decisions to the `submit_decision` topic defined by div-model `game/games.py`.
The others control setting/clearing any error status message it returns.

```jsx
import {createAction} from 'redux-actions';

import AutobahnReact from 'simpl-react/lib/autobahn';

// submit player decision then calculate result if both dividend and divisor have been submitted
export const submitDecision =
    createAction('SUBMIT_DECISION', (period, operand, ...args) =>
        AutobahnReact.call(`model:model.period.${period.id}.submit_decision`, [operand])
    );
    
// actions for setting / clearing a status message
export const setStatus = createAction('SET_STATUS');
export const clearStatus = createAction('CLEAR_STATUS');

```
**NOTE**: The `submit_decision` action calls this topic because the div-model `game/games.py` `submit_decision` endpoint registers as an RPC on the topic. 
Because `submit_decision` validates the operand, using an RPC allows us to check the returned status.

Create `js/reducers/StatusReducer.js` to handle the `setStatus` and `clearStatus` actions:

```jsx
import {createReducer} from 'redux-create-reducer';
import recycleState from 'redux-recycle';

import {recyleStateAction} from 'simpl-react/lib/actions/state';

import {
  setStatus,
  clearStatus
} from '../actions/Actions';

const initial = {message: ''};

const status = recycleState(createReducer(initial, {
  [setStatus](state, action) {
    const message = action.payload;
    console.log("setting status.message: ", message);
    return Object.assign({}, state, {
      message: message
    });
  },
  [clearStatus](state) {
    console.log("clearing status.message");
    return Object.assign({}, state, {
      message: ''
    });
  },
}), `${recyleStateAction}`);

export default status;

```

Then add `status` to `reducers/combined/appReducers.js`:

```jsx
import {simplReducers} from 'simpl-react/lib/reducers/combined';
import {reducer as form} from 'redux-form';

import status from '../StatusReducer';

const reducers = simplReducers({
  form,
  // Add your customer reducers here, if any.
  status
});

export default reducers;

```

Create a presentational component `js/components/DecisionForm.js` for entering player decisions and displaying an error message:

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

Wrap it in a container component that checks the returned status `js/containers/DecisionFormContainer.js`:

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
          } else {
            console.log("DecisionFormContainer.submitDecision succeeded");
            dispatch(setStatus(''));
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
    const {message} = this.props;
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
          <h4><span>{message}</span></h4>
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
  message: PropTypes.string,
};

PlayerHome.defaultProps = {
  message: '',
}

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
  // console.log("PlayerHome: currentPhase=", currentPhase, ", playPhase.id=", playPhase.id, ", canPlay=", canPlay);

  const other_role = state.simpl.role.find((r) => r.id !== runuser.role);
  const other_role_name = other_role.name;

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
    result,
    message: state.status.message,
  };
}

const module = connect(
  mapStateToProps,
  null
)(PlayerHome);

export default module;

```

Now, when players log in, they see a form for entering decisions for their role and a logout link:

![](/assets/img/tutorials/multi-player/Players_Home.png){: width="80%" }

As a world's players submit decisions, the redux state automatically updates with new periods, decisions and results:

![](/assets/img/tutorials/multi-player/Simpl_Players_Home.png){: width="100%" }

Log in as `s2@div.edu` with password `s2` and submit a zero decision. You should see a notification like this:

![](/assets/img/tutorials/multi-player/Status_Notification.png){: width="80%" }

Congratulations! You are now ready to build the [Multi-player Game Leader UI]({% link _docs/tutorials/multi-player/frontend-leader.md %})
