# Low-level Decorators

### publishes() decorator

`containers/Publisher.jsx`

```javascript
import {publishes} from 'simpl/lib/decorators/pubsub/publishes';

import MyComponent from '../components/MyComponent.react';

Publisher = publishes(topic, options = {})(MyComponent);

```

#### Presentational Component

`components/SendMessage.jsx`;

```jsx
import React from 'react';


class SendMessage extends React.Component {
  onPressedKey(e) {
    if (e.charCode === 13) { // Enter key
      e.preventDefault();
      const node = this.refs.msgInput;
      this.props.publish({ args: [node.value] });
      node.value = '';
    }
  }
  render() {
    return (
      <input ref="msgInput" onKeyPress={(e) => this.onPressedKey(e)} type="text" />
    );
  }
});

SendMessage.propTypes = {
  publish: React.PropTypes.func
};

export default SendMessage;
```

##### Props

* `onPublished(topic, args, kwargs, details, publicationID)`: This function will be called after new message is successfully published
* `onPublishedError(variable, args, kwargs, details, error)`: This function will be called after a failure in publishing the message.
* `publish({args: [], kwargs: {}})`: Call this method to publish a message on the topic


#### Smart Component

`SendMessageContainer.jsx`

```javascript

import { connect } from 'react-redux';

import {publishes} from 'simpl/lib/decorators/pubsub/publishes';

import SendMessage from '../components/SendMessage';

function mapStateToProps(state, ownProps) {
  return {};
}

function mapDispatchToProps(dispatch, ownProps) {
  return {
    onPublished(topic, args, kwargs, details, publicationID) {
      console.log(kwargs);
    }
  };
}

const PublisherSendMessage = publishes('com.example.sims.simpl-calc.chat')(SendMessage);

const SendMessageContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(PublisherSendMessage);

export default SendMessageContainer;

```

##### Props

* `onPublished(topic, args, kwargs, details, publicationID)`: This function will be called after new message is successfully published
* `onPublishedError(variable, args, kwargs, details, error)`: This function will be called after a failure in publishing the message.
  
### subscribes() decorator

`containers/Subscriber.jsx`

```javascript
import {subscribes} from 'simpl/lib/decorators/pubsub/subscribes';

import MyComponent from '../components/MyComponent';

Subscriber = subscribes(topics, options = {})(MyComponent);
```

Arguments:

* `topics`: Required. A string, function or an array of strings / function of topics that the componen will subscribe to. Functions will receive a `props` argument and must return a string.

#### Presentational Component

`components/Results.jsx`

```jsx
import React from 'react';


const Result = function Result(result) {
  return (
    <tr key={result.id}>
      <td>{result.data.sum}</td>
      <td>{result.data.spread}</td>
      <td>{result.data.score}</td>
      <td>{result.data.totalScore}</td>
    </tr>
  );
};


const Result = function Result(props) {
  let results;

  if (props.data !== undefined) {
    results = props.data.map(Result);
  }

  return (
    <div>
      <h3>Results</h3>
      <table>
        <thead>
          <tr>
            <td>sum</td>
            <td>spread</td>
            <td>score</td>
            <td>total score</td>
          </tr>
        </thead>
        <tbody>{results}</tbody>
      </table>
    </div>
  );
};

export default Result;
```

##### Props

* `data`: array or object. The messages received. If the smart component has `store=true` (the default), it will be an array, otherwise it will be an object with only the latest message received.

#### Smart Component

`containers/NotificationContainer.jsx`

```javascript
import { connect } from 'react-redux';

import {subscribes} from 'simpl/lib/decorators/pubsub/subscribes';

import Notification from '../components/Notification';


function mapStateToProps() {
  return {
  };
}

function mapDispatchToProps() {
  return {
    onReceived(route, args, kwargs, details) {
      console.log(arguments);
    },
  };
}

const SubscribedNotification = subscribes(
  'com.example.sims.simpl-calc.chat'
)(Notification);

const NotificationContainer = connect(
  mapStateToProps,
  mapDispatchToProps
)(SubscribedNotification);

export default NotificationContainer;


```

`containers/MyApp.jsx`

```jsx
import React from 'react';
import NotificationContainer from '../containers/NotificationContainer';

const App = function App() {
  return (
    <NotificationContainer
      onReceived={(route, args, kwargs, details) => { console.table([kwargs]) } }
    />
  );
};
```

#### Props

* `append`: boolean. Default: `true`. Defines if messages should 'pile up' or if there will only be the last message received
* `onReceived(route, args, kwargs, details)`: function. Optional. Called when a new message is received.


