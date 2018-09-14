---
title: Build the Multi-player Game Frontend UI
permalink: /docs/tutorials/multi-player/frontend/
layout: docs
description:
---

## Build the Multi-player Game Frontend

### Prerequisites

This tutorial assumes you are familiar with [react](https://reactjs.org) and [redux](https://github.com/reduxjs/react-redux).

You will need to have these installed:

* [Node](https://nodejs.org) >= 5.7.0
* [NPM](https://nodejs.org) >= 3.6.0
* Python >= 3.6
* [virtualenv](https://virtualenv.pypa.io/en/stable/)

Have the [Games API service]({% link _docs/getting-started.md %}) running on http://localhost:8100/ and 
the [div-model service]({% link _docs/tutorials/multi-player/modelservice.md %}) running on http://localhost:8080. 

### Installation

In a separate terminal, create a new virtualenv called 'div-ui':

```shell
$ mkvirtualenv div-ui
```

Install the `cookiecutter` Python package:

```shell
$ pip install cookiecutter
```

Use `cookiecutter` to create the boilerplate for your game Frontend.

```shell
$ cookiecutter https://github.com:simplworld/simpl-ui-cookiecutter.git
```

For the `game_slug` value, enter `div` (the slug value you used in the [modelservice tutorial]({% link _docs/tutorials/multi-player/modelservice.md %})). 
For all other values, you can use the default or choose your own.

For example,

```shell
project_name [Simulation UI]: Div UI	
repo_slug [div-ui]: 
project_slug [div_ui]: 
game_slug [div-ui]: div
modelservice_slug [div-model]:      
topic_root [world.simpl]: 
app_slug [frontend]: 
version [0.1.0]: 
```

After the project layout is created, install the requirements:

```shell
$ cd div-ui
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

In Chrome, head to `http://localhost:8000/` and login as `s1@div.edu` with password `s1`.
Once you are logged in, you should see the 'Hello Player' message of the skeleton app.
Open Chrome's DevTools, and select the 'Redux' tab. You will see a list of actions, and the
current `state` of the store. Note that the state has a property named `simpl`. Expand the `simpl` property
to see all the scope properties associated with the current user.

![](/assets/img/tutorials/multi-player/Hello_Simpl_Player.png){: width="100%" }

These properties will be updated as the model service adds, removes or updates scopes.
You will connect your components to the properties and they will update accordingly.

Logout by going to `localhost:8000/logout/` in your browser. Then login as `leader@div.edu` with password `leader`.
Once you are logged in, you should see the 'Hello Leader' message of the skeleton app. If you look at the `simpl`
state properties, information about all the run's worlds has been loaded.

![](/assets/img/tutorials/multi-player/Hello_Simpl_Leader.png){: width="100%" }

In a multi-player simulation, players are assigned to a world with other players.
The cookiecutter template assumes you are implementing a multi-player simulation in which players
see only their world's information. By default, leaders can see the worlds and users in their subscribed runs,
but not the scenarios of other users. To accomplish this, the template `js/modules/Root.js` sets the simpl decorator's 
`loadAllScenarios` argument false to block access to scenarios of other users.

### Implementation

To implement your UI, you will write [Container Components and Presentational Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0).

The Presentational Components will provide the necessary markup to render UI elements, while the Container Components will wrap them providing the necessary data.

First, you will [Build the Multi-player Game Player UI]({% link _docs/tutorials/multi-player/frontend-player.md %}).

Finally, you will [Build the Multi-player Game Leader UI]({% link _docs/tutorials/multi-player/frontend-leader.md %}).

This concludes our tutorial! We have barely scratched the surface. A completed example implementation is available at 
[https://github.com/simplworld/simpl-div-ui](https://github.com/simplworld/simpl-div-ui)
that uses the game slug `simpl-div`.

There's much more that you can do with Simpl. For more informations, check out the service-specific documentation.
