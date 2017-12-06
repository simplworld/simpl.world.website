# Low-level Decorators

### simpl() decorator

This decorator allows your app to stay in sync with changes from the model service implementation.

The UI will receive an action and its store will be updated every time a new Scope is added.

In order to update your state, you need to create your reducers by using the `Simpl.reducers.combined.simplReducers` function:

```javascript
import {simplReducers} from 'simpl/lib/reducers/combined';

const reducers = simplReducers({});

export default reducers;
```

You can add your own reducers to it by passing them as arguments:

```javascript
import {simplReducers} from 'simpl/lib/reducers/combined';

import addresses from '../addressReducers';
import counter from '../counterReducers';


const reducers = simplReducers({
  addresses,
  counter,
});

export default reducers;

```

Then create a component that will show the loading state of your app. This component will receive a `connectionStatus` prop that will indicate the loading progress.

`components.Progress.jsx`

```jsx
import React from 'react';

import { CONNECTION_STATUS } from '../constants';


function Progress(props) {
    let content;
    if (props.connectionStatus === CONNECTION_STATUS.CONNECTING) {
      text = 'Connecting…';
    } else if (props.connectionStatus === CONNECTION_STATUS.CONNECTED) {
      text = 'Loading data…';
    } else if (props.connectionStatus === CONNECTION_STATUS.OFFLINE) {
      text = 'Connection lost. If this persist, please contact the administrator.';
    }
    return (
        <div>{content}</div>
    );
}

Progress.propTypes = {
    connectionStatus: React.PropTypes.string.isRequired,
};

export default Connecting;
```

Last, wrap you app with the `simpl` decorator:

`containers/App.jsx`

```jsx
import React from 'react';

import {simpl} from 'simpl/lib/decorators/simpl';

import Progress from '../components/Progress';
import DecisionContainer from '../containers/DecisionContainer';


const App = function App() {
  return (
    <div>
      <h4>Decisions</h4>
      <DecisionContainer />
    </div>
  );
};

export default simpl({
  authid: USER_SIMPL_ID,
  password: 'nopassword',
  url: 'ws://localhost:8080/ws',
  topics: ['model:model.actions.game'],
  progressComponent: Progress,
  root_topic: CROSSBAR_ROOT_TOPIC,
})(App);

```

`containers/DecisionContainer.jsx`

```jsx
import React from 'react';

import { connect } from 'react-redux';


function mapStateToProps(state) {
  return {
    decisions: state.simpl.decision || [],
  };
}

function mapDispatchToProps() {
  return {
  };
}

const Decision = function Decision(decision) {
  return <li key={decision.id}>Role: {decision.role} Term: {decision.term}</li>;
};

const Decisions = React.createClass({
  render() {
    const decisions = this.props.decisions.map(Decision);

    return <ul>{decisions}</ul>
  }
})

const DecisionContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(Decisions);

export default DecisionContainer;
```
