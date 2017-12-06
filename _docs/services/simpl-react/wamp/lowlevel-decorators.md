# Low-level Decorators

### wamp() decorator

```javascript

App = wamp(MyComponent, options = {});
```

#### Presentational Component

`Counter.jsx`

```jsx
import React from 'react';

import {wamp} from 'simpl/lib/decorators/wamp';


const App = function App() {
  return (
    <div>
      <h3>My Amazing App</h3>
    </div>
  );
};

export default wamp({
  authid: USER_SIMPL_ID,
  password: 'nopassword',
  url: 'ws://localhost:8080/ws',
  prefixes: {
    model: CROSSBAR_ROOT_TOPIC,
  },
})(App);
```

##### Options

* `authid`: Optional. String or integer. The user's id on simpl-games-api.
* `password`: Optional, string. Password to use for authentication.
* `url`: Optional, string. URL to the wamp router. Defaults to `'ws://localhost:8080/ws'`
* `realm`: Optional, string. Realm to use. Defaults to `'realm1'
* `prefixes`: Optional, object. A map of topic shortcuts. You can name prefixes and use them to publish / subscribe to those topics. Eg: if you define `{model: com.example.sim.model}`, you can publish to `com.example.sim.model.action` by using `model:action`. Defaults to an empty object.
