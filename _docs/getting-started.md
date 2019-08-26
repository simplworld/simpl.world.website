---
title: Getting Started
permalink: /docs/getting_started/
layout: docs
description: You can get a taste of Simpl by running the simpl-calc example simulation.
---

## Getting Started

You can get a taste of Simpl by running the simpl-calc example simulation.

## Prerequisites

You will need to have these installed:
   * PostgreSQL >= 9.6
   * Python >= 3.6
   * [virtualenv](https://virtualenv.pypa.io/en/stable/)

If you are using the Mac OS, [Postgres.app](https://postgresapp.com) is an easy way to install and configure PostgreSQL.

If you are using Windows 10, the tutorials have been tested using Ubuntu 16.04 with the [Windows Linux Subsystem](https://docs.microsoft.com/en-us/windows/wsl/install-win10).  Note, you'll need to be comfortable installing virtualenv, Python 3.6, as well as a number of supporting OS packages in order to get simpl up and running on Windows.  

Install gulp and webpack globally to ensure they are on your PATH

```shell
sudo npm install --global gulp
sudo npm install --global webpack
```

## Run the Simpl Games API Service

Clone the simpl-games-api repository and install simpl-games-api:

```shell
git clone --branch v0.7.9 https://github.com/simplworld/simpl-games-api.git
cd simpl-games-api
add2virtualenv .
pip install -r requirements.txt
```

Create a Simpl database:

```shell
createdb simpl
./manage.py migrate
./manage.py create_simpl_user
```

Note, for Windows users, you'll need to edit the database line with the following in `config/settings/common.py` to access psql (assuming it is installed locally):

```python
DATABASES = {
    # Raises ImproperlyConfigured exception if DATABASE_URL not in os.environ
    'default': env.db("DATABASE_URL", default="postgres://simpl:simpl@localhost:5432/simpl"),
}
```

Start the simpl-games-api web service:

```shell
./manage.py runserver 0.0.0.0:8100
```

## Run the Simpl Calc Model Service

In a separate terminal, clone the simpl-calc-model repository and install simpl-calc model:

```shell
git clone https://github.com/simplworld/simpl-calc-model.git
cd simpl-calc-model
add2virtualenv .
pip install -r requirements.txt
```

Add the simp-calc game to the Simpl database along with some test users:

```shell
./manage.py create_default_env
```

Start the model service by running:

```shell
./manage.py run_modelservice
```

By default the service will bind to `0.0.0.0:8080`.


## Run the Simpl Calc Frontend UI

In a separate terminal, clone the simpl-calc-ui repository and install simpl-calc:

```shell
git clone https://github.com/simplworld/simpl-calc-ui.git
cd simpl-calc-ui
mkvirtualenv simpl-calc-ui
add2virtualenv .
pip install -r requirements.txt
./manage.py migrate
```

Start your frontend webserver with:

```shell
./manage.py runserver 0.0.0.0:8000
```

In a separate terminal, update node_modules and run Gulp to compile JS and SASS

```shell
npm install
gulp
```

## Using the Simpl Calc Simulation

The simpl-cal simulation is now running at `http://localhost:8000/`

Log in as player **s1@calc.edu** (password **s1**) or player **s2@calc.edu** (password **s2**). Once logged in, use the simulation to add numbers to a total.

In another browser, log in as **leader@calc.edu** (password **leader**) to see the player totals update as they change over time.
