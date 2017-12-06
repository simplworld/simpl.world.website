# High-level Components

### RPCCAller

This component will call a remote procedure and dispatch an action.

To use it, wrap it around your presentational component. The function `this.props.call({args, kwargs})` will be passed down, and you can use it to call the procedure. Chain to this function to receive the result.

`Counter.jsx`

```jsx
import React from 'react';


const Counter = function Counter(props) {
  return (
    <div>
      <h3>Call to the model-service</h3>
      <p>
        <input
          ref="term"
          name="term"
          type="number"
          defaultValue={props.counter.term}
        />

        <button onClick={() => this.props.call({ term: parseInt(this.refs.term.value, 10) })}>Send</button>
      </p>
    </div>
  );
});

export default Counter;

```

`containers/CounterContainer.jsx`

```javascript
import { connect } from 'react-redux';

import { updateCounter } from '../actions/CounterActions';
import Counter from '../components/Counter';


function mapStateToProps(state) {
  return {counter: state.counter};
}

function mapDispatchToProps(dispatch, ownProps) {
  return {
    call(...args) {
      return ownProps.call(...args).then((result) => {
        return dispatch(updateCounter(result));
      })
    },
  };
}

const CounterContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(Counter);

export default CounterContainer;
```

`containers/App.jsx`

```jsx
import React from 'react';

import RPCCaller from 'simpl/lib/components/RPCCaller.react';

import CounterContainer from '../containers/CounterContainer';


const App = function App() {
  return (
    <div>
      <RPCCaller callee="com.example.sims.simpl-calc.model.actions.play_term">
        <CounterContainer />
      </RPCCaller>
    </div>
  );
};

export default App;
```

#### Props

* `callee`: string. Required. The callee to call.
* `options`: object. Optional. The [call options](http://autobahn.ws/python/reference/autobahn.wamp.html#autobahn.wamp.CallOptions) that will be used. Defaults to `{}`.
