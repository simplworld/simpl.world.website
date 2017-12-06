# Low-level Decorators

### calls() decorator

```js
import {calls} from 'simpl/lib/decorators/rpc/calls'


Caller = calls(procedure, options = {})(MyComponent);
```

#### Presentational Component

`Counter.jsx`

```jsx
import React from 'react';


class Counter extends React.Component {
  onClick() {
    this.props.call({ kwargs: { term: parseInt(this.refs.term.value, 10) } })
  }
  render() {
    return (
      <div>
        <h3>Call to the model-service</h3>
        <p>
          <input
            ref="term"
            name="term"
            type="number"
            defaultValue={this.props.counter.term}
          />

          <button onClick={() => this.onClick()}>Send</button>
        </p>
      </div>
    );
  }
});

export default Counter;
```

##### Props

* `call(topic, args, kwargs, details, publicationID)`: This function will be called after new message is successfully published


#### Smart Component

`CounterContainer.jsx`

```jsx
import React from 'react';
import { connect } from 'react-redux';
import {calls} from 'simpl/lib/decorators/rpc/calls'

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
      });
    },
  };
}

const _CounterContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(Counter);

CounterContainer = calls(
  'com.example.sims.simpl-calc.model.actions.play_term'
)(_CounterContainer);

export default CounterContainer;
```
