---
title: Build the Single-player Game Frontend UI
permalink: /docs/tutorials/single-player/frontend/
layout: docs
description:
---

## Build the Single-player Game Frontend

### Prerequisites

This tutorial assumes you are familiar with [react](https://reactjs.org) and [redux](https://github.com/reduxjs/react-redux).

You will need to have these installed:

* [Node](https://nodejs.org) >= 5.7.0
* [NPM](https://nodejs.org) >= 3.6.0
* Python >= 3.6
* [virtualenv](https://virtualenv.pypa.io/en/stable/)

Have the [Games API service]({% link _docs/getting-started.md %}) running on http://localhost:8100/ and 
the [calc-model service]({% link _docs/tutorials/single-player/modelservice.md %}) running on http://localhost:8080. 

### Installation

In a separate terminal, create a new virtualenv called 'calc-ui':

```shell
$ mkvirtualenv calc-ui
```

Install the `cookiecutter` Python package:

```shell
$ pip install cookiecutter
```

Use `cookiecutter` to create the boilerplate for your game Frontend.

```shell
$ cookiecutter https://github.com:simplworld/simpl-ui-cookiecutter.git
```

For the `game_slug` value, enter `calc` (the slug value you used in the [modelservice tutorial]({% link _docs/tutorials/single-player/modelservice.md %})). 
For all other values, you can use the default or choose your own.

For example,

```shell
project_name [Simulation UI]: Calc UI
repo_slug [calc-ui]:
project_slug [calc_ui]:
game_slug [calc-ui]: calc
modelservice_slug [calc-model]:
topic_root [world.simpl]:
app_slug [frontend]:
version [0.1.0]:
```

After the project layout is created, install the requirements:

```shell
$ cd calc-ui
$ pip install -r requirements.txt
```

In a separate terminal, install the project's JavaScript node modules and run gulp to keep the web server's Javascript updated as you work on the frontend:

```shell
$ npm install
$ gulp
```

To aid development, you can install the following Chrome DevTools Extensions:

* [Redux DevTools Extension](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)
* [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)

It is also recommended you configure your editor to integrate with ESLint:

* [PyCharm](https://www.jetbrains.com/help/pycharm/2016.1/eslint.html)
* [SublimeText2](https://github.com/roadhump/SublimeLinter-eslint)

### Configuration

Like most websites, the frontend service will need a place where it can store information about sessions and their users. 
The user's specific information will be fetched from the [Simple Games API]({% link _docs/getting-started.md %}) and kept in sync automatically.

**NOTE** For the purposes of this tutorial we're going to use SQLite, but this can be changed to match whatever database backend you prefer. 
In a production environment, you'll likely want to switch to something like PostgreSQL.

First, let's create the necessary local tables:

```shell
$ ./manage.py migrate
```

Then, start your frontend service with:

```shell
$ ./manage.py runserver 0.0.0.0:8000
```

In Chrome, head to `http://localhost:8000/` and login as `s1@calc.edu` with password `s1`.
Once you are logged in, you should see the 'Hello Player' message of the skeleton app.

![](/assets/img/tutorials/single-player/Hello_Player.png)

Open Chrome's DevTools, and select the 'Redux' tab. You will see a list of actions, and the
current `state` of the store. Note that the state has a property named `simpl`. Expand the `simpl` property
to see all the scope properties associated with the current user.

![](/assets/img/tutorials/single-player/Hello_Simpl_Player.png){: width="100%" }

These properties will be updated as the model service adds, removes or updates scopes.
You will connect your components to the properties and they will update accordingly.

Next, logout by going to `localhost:8000/logout/` in your browser. Then login as `leader@calc.edu` with password `leader`.
Once you are logged in, you should see the 'Hello Leader' message of the skeleton app. If you look at the `simpl`
state properties, you'll see all the run's runusers have been loaded. No player scenarios have been loaded.

![](/assets/img/tutorials/single-player/Hello_Simpl_Leader1.png){: width="100%" }

In a multi-player simulation, players are assigned to a world with other players.
The cookiecutter template assumes you are implementing a multi-player simulation in which players
see only their own and their world's data. By default, leaders can see the worlds and users in their subscribed runs,
but not the scenarios of other users. Consequently, the template `js/modules/Root.js` sets the simpl decorator's 
`loadAllScenarios` argument false to block access to scenarios of other users.

We want Calc leaders to have access to all information on all players in their runs. Since Calc players are not associated with
a world, we need to modify the template `js/modules/Root.js` code to load all player scenarios for leaders.

In your `js/modules/Root.js`:

```jsx
export default simpl({
  authid: AUTHID,
  password: 'nopassword',
  url: `${MODEL_SERVICE}`,
  progressComponent: Progress,
  root_topic: ROOT_TOPIC,
  topics: () => topics,
  loadAllScenarios: false
})(RootContainer);
```

change the simpl decorator's `loadAllScenarios` argument to LEADER:

```jsx
export default simpl({
  authid: AUTHID,
  password: 'nopassword',
  url: `${MODEL_SERVICE}`,
  progressComponent: Progress,
  root_topic: ROOT_TOPIC,
  topics: () => topics,
  loadAllScenarios: LEADER
})(RootContainer);
```

Refresh your Chrome browser page and you'll see all the run's player scenarios have been loaded into the `simpl` state.

![](/assets/img/tutorials/single-player/Hello_Simpl_Leader2.png){: width="100%" }

### Implementation

To implement your UI, you will write [Container Components and Presentational Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0).

The Presentational Components will provide the necessary markup to render UI elements, while the Container Components will wrap them providing the necessary data.

First, you will [Build the Single-player Game Player UI]({% link _docs/tutorials/single-player/frontend-player.md %}).

Finally, you will [Build the Single-player Game Leader UI]({% link _docs/tutorials/single-player/frontend-leader.md %}).

This concludes our tutorial! We have barely scratched the surface. A completed example implementation is available at 
[https://github.com/simplworld/simpl-calc-ui](https://github.com/simplworld/simpl-calc-ui)
that uses the game slug `simpl-calc`.

There's much more that you can do with Simpl. For more informations, check out the service-specific documentation.
