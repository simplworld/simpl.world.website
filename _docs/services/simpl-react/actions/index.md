# Actions

To create an action, you can use the `createAction` convenience function from [`redux-actions`](https://github.com/acdlite/redux-actions).

`actions/myactions.js`
```js
import { createAction } from 'redux-actions';


export const myAction = createAction('MY_ACTION');
```

The function will create an action that serializes to the passed action name, and can be directly imported and used as a key in a reducer:


`reducers/myreducer.js`
```js
import { createReducer } from 'redux-create-reducer';
import recycleState from 'redux-recycle';

import { recyleStateAction } from 'simpl/lib/actions/state';

import { myAction } from '../actions/myactions';


const simpl = recycleState(createReducer(initial, {
  [myAction](state, action) {
    // update state accordingly...
    return Object.assign({}, state, action.payload);
  },
}), `${recyleStateAction}`);
```

## PubSub

To create an action that publishes to a WAMP topic, you should use the Autobahn client included with `simpl`:

`actions/myactions.js`
```js
import { createAction } from 'redux-actions';
import AutobahnReact from 'simpl/lib/autobahn';


export const myAction = createAction('MY_ACTION', (some_scope, ...args) =>
    AutobahnReact.publish(`model:model.${some_scope.resource_name}.${some_scope.id}.topic_name`, args)
);
```

Since publishing to a topic does not return any value, and "completes" before anything is done on the server side, there is very little reason to catch the action in a reducer.

## RPC

To create an action that calss a WAMP topic, you should use the Autobahn client included with `simpl`:

`actions/myactions.js`
```js
import { createAction } from 'redux-actions';
import AutobahnReact from 'simpl/lib/autobahn';


export const getSomeData = createAction('GET_SOME_DATA', (scope, ...args) => (
  AutobahnReact.call(`${scope}.get_some_data`, args)
));
```

The action's `payload` will contain the value returned by the remote procedure:

`reducers/myreducer.js`
```js
import { createReducer } from 'redux-create-reducer';
import recycleState from 'redux-recycle';

import { recyleStateAction } from 'simpl/lib/actions/state'
import { getSomeData } from '../actions/myactions';


const simpl = recycleState(createReducer(initial, {
  [getSomeData](state, action) {

    return Object.assign({}, state, action.payload);
  },
}), `${recyleStateAction}`);
```

### Chain RPC promises

The action returns a promise which will be resolved with the returned value. This means tha t you can chain to the action to dispatch further actions after the call completes:

`MyContainer.react.js`
```js
import { connect } from 'react-redux';
import { getSomeData, someOtherAction } from '../actions/myactions';

import { MyComponent } from '../components/MyComponent.react';


function mapDispatchToProps(dispatch) {
    return {
        myHandler: function(someValue) {
            dispatch(getSomeData(someValue)).then(
                (callResult) => dispatch(someOtherAction(callResult))
            )
        }
    };
}

const MyContainer = connect(
    null,
    mapDispatchToProps
)(MyComponent);

export default MyContainer;
```