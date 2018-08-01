---
title: Build the Single-player Game Leader UI
permalink: /docs/tutorials/single-player/frontend-leader/
layout: docs
description:
---

## Build the Single-player Game Leader UI

We want leaders to be able see the player results. You will now update the leader home page so they can.

Create a presentation component `js/components/PlayerResultRow.js` for displaying one player's results:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

function PlayerResultRow(props) {
  return (
    <tr>
      <td>{props.runuser.email}</td>
      <td>{props.periodsPlayed}</td>
      <td>{props.total}</td>
    </tr>
  );
}

PlayerResultRow.propTypes = {
  runuser: PropTypes.object.isRequired,
  periodsPlayed: PropTypes.number.isRequired,
  total: PropTypes.number.isRequired
};

export default PlayerResultRow;
```

wrap it in a container component `js/containers/PlayerResultRowContainer.js`

```jsx
import {connect} from 'react-redux';
import {withRouter} from 'react-router';

import PlayerResultRow from '../components/PlayerResultRow';

function mapStateToProps(state, ownProps) {
  const runuser = ownProps.runuser;

  const scenario = state.simpl.scenario.find(
    (s) => runuser.id === s.runuser
  );

  let periodsPlayed = 0;
  let total = 0;
  if (scenario) { // avoid runtime errors while state is loading
    const unsortedPeriods = state.simpl.period.filter(
      (p) => scenario.id === p.scenario
    );
    const periods = _.sortBy(unsortedPeriods, (p) => p.order);
    const periodOrder = _.last(periods).order;

    if (periodOrder > 1) { // pull total from last result
      const lastPeriod = periods[periodOrder - 2];
      periodsPlayed = lastPeriod.order;

      const lastResult = state.simpl.result.find(
        (s) => lastPeriod.id === s.period
      );
      total = lastResult.data.total;
    }
  }

  return {
    runuser,
    periodsPlayed,
    total
  };
}

const PlayerResultRowContainer = connect(
  mapStateToProps,
  null
)(PlayerResultRow);

export default withRouter(PlayerResultRowContainer);
```

In your `js/modules/LeaderHome.js`, replace the original contents with:

```jsx
import React from 'react';
import PropTypes from 'prop-types';

import {connect} from 'react-redux';

import PlayerResultRowContainer from '../containers/PlayerResultRowContainer'

class LeaderHome extends React.Component {

  render() {
    const name = this.props.runuser.first_name + ' ' + this.props.runuser.last_name;
    const playerRows = this.props.players.map(
      (p) => <PlayerResultRowContainer key={p.id} runuser={p}/>
    );
    return (
      <div>
        <div>
          <h1>Hello {name}</h1>
        </div>
        <div>
          <table>
            <thead>
            <tr>
              <th width="30%"> Player</th>
              <th width="30%"> Periods Played</th>
              <th width="30%"> Total</th>
            </tr>
            </thead>
            <tbody>
            {playerRows}
            </tbody>
          </table>
        </div>
        <br/>
        <a href="/logout/" className="btn btn-success btn-lg">Logout</a>
      </div>
    );
  }
}

LeaderHome.propTypes = {
  runuser: PropTypes.object.isRequired,
  players: PropTypes.array.isRequired
};

function mapStateToProps(state) {
  const runuser = state.simpl.current_runuser;

  const unsortedPlayers = state.simpl.runuser.filter(
    (ru) => runuser.id !== ru.id
  );
  const players = _.sortBy(unsortedPlayers, (p) => p.email);

  return {
    runuser,
    players
  };
}

const module = connect(
  mapStateToProps,
  null
)(LeaderHome);

export default module;
```

Now when a leader logs in, they see the current player results:

![](/assets/img/tutorials/single-player/Leader_Home.png)

Let's add some styling to make it easier to read the table of results.

In `frontend/templates/frontend.home.html`, replace

```html
  <head>
  </head>
```

with

```html
  <head>
    <style>
    table, th, td { border: 1px solid black;  }
    </style>
  </head>
```

![](/assets/img/tutorials/single-player/Leader_Home2.png)

To see the revised leader page in action, open an incognito window and login into http://localhost:8000/ as `s2@calc.edu` with password `s2`.

![](/assets/img/tutorials/single-player/Simpl_Play.png){: width="100%" }

Submit a decision in the `s2@calc.edu` window. The simpl state in both browser windows will update with a new result causing the leader home page to update accordingly.

![](/assets/img/tutorials/single-player/Simpl_Play2.png){: width="100%" }
