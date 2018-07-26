---
title: Single Player Tutorial
permalink: /docs/getting_started/
layout: docs
description: You can get a taste of Simpl by running the simpl-calc example simulation.
---

## Simpl Getting Started Guide

You can get a taste of Simpl by running the simpl-calc example simulation.

## Prerequisites

These instructions assume you are working in the Simpl Vagrant environment:

* [Simpl Python-Dev Development Environment (CentOS 7) Vagrant Box](https://github.com/simplworld/python-vagrant-centos7)
    * PostgreSQL >= 9.6
    * Python >= 3.6

Clone and run the Simpl `python-vagrant-centos7` image:

```shell
$ git clone git@github.com:simplworld/python-vagrant-centos7.git
$ cd python-vagrant-centos7
$ vagrant up
```


## Run the Simpl Games API Service

Clone the simpl-games-api repository in the Vagrant image's project directory:

```shell
$ cd python-vagrant-centos7/projects
$ git clone git@github.com:simplworld/simpl-games-api.git
```

Connect to the Vagrant environment and install simpl-games-api:

```shell
$ vagrant ssh
$ mkvirtualenv simpl-games-api
$ cd projects/simpl-games-api
$ add2virtualenv .
$ pip install -r requirements.txt
```

Create a Simpl database:

```shell
$ createdb simpl
$ ./manage.py migrate
$ ./manage.py create_simpl_user
```

Start the simpl-games-api web service:

```shell
$ ./manage.py run_gunicorn
```


## Run the Simpl Calc Model Service

In a separate terminal, clone the simpl-calc-model repository in the Vagrant image's project directory:

```shell
$ cd python-vagrant-centos7/projects
$ git clone git@github.com:simplworld/simpl-calc-model.git
```

Connect to Vagrant environment and install the simpl-calc Model Service:

```shell
$ vagrant ssh
$ mkvirtualenv simpl-calc-model
$ cd projects/simpl-calc-model
$ add2virtualenv .
$ PIP_PROCESS_DEPENDENCY_LINKS=1 pip install -r requirements.txt
```

Add the simp-calc game to the Simpl database along with some test users:

```shell
$ ./manage.py create_default_env
```

Start service by running:

```shell
$ ./manage.py run_modelservice
```

By default the service will bind to `0.0.0.0:8080`.


## Run the Simpl Calc Frontend UI

In a separate terminal, clone the simpl-calc-ui repository in the Vagrant image's project directory:

```shell
$ cd python-vagrant-centos7/projects
$ git clone git@github.com:simplworld/simpl-calc-ui.git
```

Connect to Vagrant environment and install the simpl-calc Front End

```shell
$ vagrant ssh
$ mkvirtualenv simpl-calc-ui
$ cd projects/simpl-calc-ui
$ add2virtualenv .
$ pip install -r requirements.txt
$ ./manage.py migrate
```

Start your frontend webserver with:

```shell
$ ./manage.py runserver 0.0.0.0:8000
```

Install gulp and/or webpack globally outside Vagrant to ensure they are on your PATH

```shell
$ sudo npm install --global webpack
$ sudo npm install --global gulp
```

In a separate terminal, update node_modules and run Gulp to compile JS and SASS

```shell
$ cd to simpl-calc-ui directory
$ npm install
$ gulp
```

## Using the Simpl Calc Simulation

The simpl-cal simulation is now running at `http://localhost:8000/`

Log in player s1@calc.edu (password s1) or player s2@calc.edu (password s2). Once logged in, use the simulation to add numbers to a total.

In another browser, log in as leader@calc.edu (password leader) to see the player totals update over time.
