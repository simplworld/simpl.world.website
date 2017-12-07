---
title: Building the Frontend UI
permalink: /docs/tutorials/frontend/
layout: docs
description:
---

## Building the Frontend UI

### Prerequisites

This tutorial assumes you already the following software installed:

* Python >= 3.4
* Node >= 5.7.0
* NPM >= 3.6.0

!!! note
    This tutorial assumes you still have both the [Games API service](./games-api.md) (http://localhost:8100/) and the [Model service](./games-api.md) (http://localhost:8080/) running.  Please open up an additional terminal window and SSH into the Vagrant box before continuing.

### Installation

First, create a new virtualenv called 'simpl-calc-ui':

```bash
$ mkvirtualenv simpl-calc-ui
```

Then, install the `cookiecutter` Python package:

```bash
$ pip install cookiecutter
```

Change to the `/vagrant/projects` folder:

```bash
$ cd projects
```

Use `cookiecutter` to create the boilerplate for your app

```bash
$ cookiecutter ssh://git@github.com:simplworld/simpl-ui-cookiecutter.git --checkout develop
```

Make sure the value for `game_slug` is the same slug you used in the [Simpl Games API](./games-api.md)
and the [modelservice tutorial](./games-api.md). If you are using defaults from the tutorial,
then the slug should be `simpl-calc`. For all other values you can use the default for the project, or choose your own.

For example,
```
project_name [Simulation UI]: SIMPL calc UI
repo_slug [simpl-calc-ui]:
project_slug [simpl_calc_ui]:
game_slug [simpl-calc-ui]: simpl-calc
modelservice_slug [simpl-calc-model]:
app_slug [frontend]:
version [0.1.0]:
```

After the project layout is created, `cd` into your repo directory and install the requirements:

```bash
$ pip install -r requirements.txt
```

Outside of Vagrant, `cd` into your your repo directory, install the JavaScript node modules and run gulp to keep the web server's javascript updated as you work on the frontend:

```bash
$ npm install
$ gulp
```

In order to aid development, you should also install the following Chrome DevTools Extensions:

* [Redux DevTools Extension](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)
* [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)

It is also recommended to configure your editor to integrate with ESLint:

* [PyCharm](https://www.jetbrains.com/help/pycharm/2016.1/eslint.html)
* [SublimeText2](https://github.com/roadhump/SublimeLinter-eslint)

### Configuration

Just like most Websites, the frontend service will need a place where it can store information about sessions and their users. The users' specific information will be fetched from the [Simple Games API](./games-api.md) and kept in sync automatically.

!!! note
    For the purposes of this tutorial we're going to use SQLite, but this can be changed to match whatever database backend you prefer. In a production environment, you'll likely want to switch to something like PostgreSQL.

First, let's create the necessary local tables:

```bash
$ ./manage.py makemigrations
$ ./manage.py migrate
```

Then, start your frontend service with:

```bash
$ ./manage.py runserver 0.0.0.0:8000
```

Now head to `http://localhost:8000/` and login as 's1@calc.edu' with passowrd 's1'. Once you are logged in, you should the 'Hello World' message of the skeleton app.

### To be continued...
