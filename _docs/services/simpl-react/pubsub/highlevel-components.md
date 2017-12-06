# High-level Components

### TopicPublisher

This component will publish messages to the specified topic.

To use it, wrap it around your presentational component. The function `this.props.publish` will be passed down, and you can use it to publish to the topic.

`SendMessage.jsx`

```jsx
import React from 'react';


class SendMessage extends React.Component {
  onPressedKey(e) {
    if (e.charCode === 13) { // Enter key
      e.preventDefault();
      let node = this.refs.msgInput;
      this.props.publish({ args: [node.value] });
      node.value = '';
    }
  }
  render() {
    return (
      <input ref="msgInput" onKeyPress={() => this.onPressedKey()} type="text" />
    );
  }
});

SendMessage.propTypes = {
  publish: React.PropTypes.func,
};

export default SendMessage;
```

`App.jsx`

```jsx
import React from 'react';

import TopicPublisher from 'simpl/lib/components/TopicPublisher.react';

import SendMessage from '../components/SendMessage';


const App = function App() {
  return (
    <div>
      <h3>Send messages</h3>
      <TopicPublisher topic="com.example.sims.simpl-calc.chat">
        <SendMessage />
      </TopicPublisher>
    </div>
  );
};

export default App;
```

#### Props

* `topic`: string. Required. The topic to publish messages to.
* `options`: object. Optional. The [publishing options](http://autobahn.ws/python/reference/autobahn.wamp.html?highlight=publishoptions#autobahn.wamp.PublishOptions) that will be used. Defaults to `{acknowledge: true, disclose_me: true, exclude_me: false}`
* `onPublished(topic, args, kwargs, details, publicationID)`: This function will be called after new message is successfully published
* `onPublishedError(topic, args, kwargs, details, error)`: This function will be called after a failure in publishing the message.


### TopicSubscriber

`Notifications.jsx`

```jsx
import React from 'react';

const Notification = function Notifications(message) {
  return (<li key={message.kwargs.publication}>{message.args[0]}</li>)
};

const Notifications = function Notifications(props) {
  let messages;

  if (props.data !== undefined) {
    messages = props.data.map(Notification);
  }

  return (
    <div>
      <ul>{messages}</ul>
    </div>
  );
};

export default Notifications;
```

`App.jsx`

```jsx
import React from 'react';

import TopicPublisher from 'simpl/lib/components/TopicPublisher.react';

import Notification from '../components/Notification';


const App = function App() {
  return (
    <div>
      <h3>Chat</h3>
      <TopicSubscriber topic="com.example.sims.simpl-calc.chat">
        <Notification />
      </TopicSubscriber>
    </div>
  );
};

export default App;
```

#### Props

* `topic`: string. Required. The topic you want the component to subscribe to.
* `append`: boolean. Default: `true`. Defines if messages should 'pile up' or if there will only be the last message received
* `options`: object. Optional. The [subscription options](http://autobahn.ws/python/reference/autobahn.wamp.html#autobahn.wamp.SubscriptionOptions) that will be used. Defaults to `{match: 'prefix'}`.
* `onReceived(route, args, kwargs, details)`: function. Optional. Called when a new message is received.

