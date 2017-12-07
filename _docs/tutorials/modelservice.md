---
title: Building the Model Service implementation
permalink: /docs/tutorials/modelservice/
layout: docs
description:
---

## Building the Model Service implementation

### Prerequisites

This part of the tutorial is a continuation of the [Simpl Games API Setup](./games-api.md).  Please complete that section before continuing.  Note, this tutorial assumes you are going through this guide on the same Vagrant box you provisioned in the previous step.

!!! note
    This tutorial assumes you still have Games API service running (http://localhost:8100/).  Please open up an additional terminal window before continuing.

### Installation

First, log into Vagrant and create a new virtualenv called 'simpl-calc-model':

```bash
$ vagrant ssh
$ mkvirtualenv simpl-calc-model
```

Install Django

```bash
$ pip install Django==1.11
```

Change to the projects folder:

```bash
$ cd projects
```

Create a Django project folder and rename it to serve as a git repository

```bash
$ django-admin startproject simpl_calc_model
$ mv simpl_calc_model simpl-calc-model
$ add2virtualenv /vagrant/projects/simpl-calc-model
```

Change to the project folder:

```bash
$ cd simpl-calc-model
```

Create a requirements.txt file that installs the simpl-modelservice and unit testing apps

```
https://github.com:simplworld/simpl-modelservice/repository/archive.zip
django-click==1.2.0

# tests
django-test-plus==1.0.17
pytest-cov==2.4.0
pytest-django==3.1.2
```

Install these requirements

```bash
$ pip install -r requirements.txt
```

Please note, if `DJANGO_SETTINGS_MODULE` is leftover from a previous session, you may need to unset it:

```bash
$ unset DJANGO_SETTINGS_MODULE
```

Create a django app that will contain your game logic:

```bash
$ ./manage.py startapp game
```

Add the following to your `INSTALLED_APPS` in `simpl_calc_model/settings.py`:

```python

INSTALLED_APPS += [
    'modelservice',
    'rest_framework',

    'game',
]

SIMPL_GAMES_URL = os.environ.get('SIMPL_GAMES_URL', 'http://localhost:8100/apis')

SIMPL_GAMES_AUTH = ('vagrant@wharton.upenn.edu', 'vagrant')

ROOT_TOPIC = 'edu.upenn.sims.simpl-calc'

CALLBACK_URL = os.environ.get('CALLBACK_URL', 'http://{hostname}:{port}/callback')
```

It's higly recommended that you set a `'users'` cache. Since the modelservice will run single-threaded, you can take advantage of the `locmem` backend:

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',
    },
    'users': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'users',
    }
}
```

### Implementation

For simplicity, we're going to create a single player Game in which each player has a Scenario that can advance multiple periods.

In your `game` app module, define our model in model.py

```
class Model(object):
    """
    The model adds an operand to the previous total and returns the result.
    """

    def step(self, operand, prev_total=0.0):
        """
        Parameters:
            operand - current period's decision
            prev_total - the calculated total from the previous period
        Returns new total
        """
        return operand + prev_total
```

In your `game` app module, add a model unit test tests/test_model.py

```
import pytest
from test_plus.test import TestCase

from game.model import Model


class ModelTestCase(TestCase):
    def setUp(self):
        self.m = Model()

    def test_create(self):
        m = Model()
        self.assertNotEqual(m, None)

    def test_first_step(self):
        m = Model()
        total = m.step(5)
        self.assertEquals(total, 5)

    def test_increase_step(self):
        m = Model()
        total = m.step(5, 3)
        self.assertEquals(total, 8)

    def test_decrease_step(self):
        m = Model()
        total = m.step(5, -2.5)
        self.assertEquals(total, 2.5)
```

Run your unit test

```
$ export DJANGO_SETTINGS_MODULE=simpl_calc_model.settings
$ py.test
```

In your `game` app module, create a management command to create our game with one run, a leader and 2 players.
1. Create and initialize CALC game.
1. Create a "default" CALC run.
1. Set the run phase to "Play".
1. Add 1 leader ("leader") to the run
1. Add 2 players with private scenar("s1", "s2") to the run.
1. Add a scenario and period 1 for each player.

```
See simpl-calc-model/game/management/command/create_default_env.py
```

Every player's move will be a `Decision` made in the current `Period`, the model will produce a `Result` for the current `Period`, and the `Scenario` will step to the next `Period`.

In your `game` app module, create a file called `runmodel.py`. Add a step_scenario function that performs these functions.

```
from modelservice.simpl import games_client
from .model import Model


def step_scenario(scenario_id, role_id):
    """
    Step the scenario's current period
    """
    periods = games_client.periods.filter(scenario=scenario_id)
    period_count = len(periods)
    period = periods[period_count - 1]

    operand = 0.0
    period_decisions = games_client.decisions.filter(period=period.id)
    if len(period_decisions) > 0:
        operand = float(period_decisions[0].data["operand"])

    prev_total = 0.0
    if period_count > 1:
        prev_period = periods[period_count - 2]
        prev_period_results = \
            games_client.results.filter(period=prev_period.id)
        if len(prev_period_results) > 0:
            prev_total = float(prev_period_results[0].data["total"])

    # step model
    model = Model()
    total = model.step(operand, prev_total)
    data = {"total" : total}

    result = games_client.results.get_or_create(
        period=period.id,
        name='results',
        role=role_id,
        data=data
    )

    # prepare for next step by adding a new period
    next_period_order = period.order + 1
    next_period = games_client.periods.get_or_create(
        scenario=scenario_id,
        order=next_period_order,
    )
    next_period.save()

    return next_period.id
```


In your `game` app module, create a file called `games.py`. **NOTE:** if you
want to use a filename other than `games.py` you must ensure the file is imported
somewhere, usually in a `__init__.py` somewhere for the `@game` decorator to find
and register your game into the system.

```
from modelservice.games import Period, Game
from modelservice.games import subscribe, register
from modelservice.games import ScopeNotFound

from .runmodel import step_scenario


class SimplCalcPeriod(Period):
    @subscribe
    def submit_decision(self, operand, **kwargs):
        """
        Receives the operand played and stores as a ``Decision`` then
        steps the model saving the ``Result``. A new ``Period`` is added to
        scenario in preparation for the next decision.
        """
        # Call will prefix the ROOT_TOPIC
        # "edu.upenn.sims.simpl-calc.model.period.1.submit_decision"

        for k in kwargs.keys():
            self.session.log.info("step: Key: {}".format(k))

        user = kwargs['user']
        role = self.get_role(user.role)

        self.add_new_decision(
            {"name": "decision",
             "data": {"operand": operand},
             "role": role.pk})

        step_scenario(self.scenario.pk, role.pk)

Game.register('simpl-calc', [
    SimplCalcPeriod,
])
```

You can start your model service by running:

```bash
$ export DJANGO_SETTINGS_MODULE=simpl_calc_model.settings
$ ./manage.py run_modelservice
```

By default the service will bind to `0.0.0.0:8080`.

This concludes the tutorial on Model Service. You can now head over to the [Frontend tutorial](./frontend.md).
