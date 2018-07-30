---
title: Build the Single-player Game Model Service
permalink: /docs/tutorials/single-player/modelservice/
layout: docs
description:
---

## Build the Single-player Game Model Service

###  Prerequisites

You will need to have these installed:
   * PostgreSQL >= 9.6
   * Python >= 3.6
   * [virtualenv](https://virtualenv.pypa.io/en/stable/)

Have the [Games API service]({% link _docs/getting-started.md %}) is running on http://localhost:8100/.  

###  Installation

In a separate terminal, create a new virtualenv called 'calc-model':

```shell
$ mkvirtualenv calc-model
```

Install Django

```shell
$ pip install Django~=1.11
```

Create a Django project folder and rename it to serve as a git repository

```shell
$ django-admin startproject calc_model
$ mv calc_model calc-model
```

Change to the project folder:

```shell
$ cd calc-model
$ add2virtualenv .
```

Create a `requirements.txt` file that installs the simpl-modelservice and unit testing apps:

```ini
https://github.com:simplworld/simpl-modelservice/repository/archive.zip

# tests
pytest==3.1.3
pytest-cov==2.5.1
pytest-django==3.1.2
django-test-plus==1.0.22
```

Install these requirements along with their dependencies:

```shell
$ PIP_PROCESS_DEPENDENCY_LINKS=1 pip install -r requirements.txt
```

Please note, if `DJANGO_SETTINGS_MODULE` is leftover from a previous session, you may need to unset it:

```shell
$ unset DJANGO_SETTINGS_MODULE
```

Create a django app that will contain your game logic:

```shell
$ ./manage.py startapp game
```

Add the following to your `INSTALLED_APPS` in `calc_model/settings.py`:

```python
INSTALLED_APPS += [
    ...

    'modelservice',
    'rest_framework',

    'game',
]

CALLBACK_URL = os.environ.get('CALLBACK_URL', 'http://{hostname}:{port}/callback')

SIMPL_GAMES_URL = os.environ.get('SIMPL_GAMES_URL', 'http://localhost:8100/apis')

SIMPL_GAMES_AUTH = ('simpl@simpl.world', 'simpl')

ROOT_TOPIC = 'world.simpl.sims.calc'
```

It's highly recommended that you set a `'users'` cache. Since the modelservice will run single-threaded, you can take advantage of the `locmem` backend:

```python
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

###  Implementation

For simplicity, we're going to create a single-player Game in which each player has a Scenario that can advance multiple periods.

In your `game` app module, define our model in `model.py`:

```python
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

In your `game` app module, add a unit test directory `tests` and a model unit test `tests/test_model.py`:

```python
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

Run your unit test:

```shell
$ export DJANGO_SETTINGS_MODULE=calc_model.settings
$ py.test
```

Create a management command that will create your game and initialize it with one run, a leader and 2 players.

Create a 'management' folder in the `game` folder and add an empty `__init__.py` file.

Create a 'commands' folder in the `management` folder  and add an empty `__init__.py` file.

Finally, create a `create_default_env.py` script in the 'commands' folder containing this code:

```python
import djclick as click

from modelservice.simpl import games_client
from modelservice.utils.asyncio import coro


def echo(text, value):
    click.echo(
        click.style(text, fg='green') + '{0}'.format(value)
    )


async def delete_default_run(api_session):
    """ Delete default Run """
    echo('Resetting the Calc game default run...', ' done')
    runs = await api_session.runs.filter(game_slug='calc')
    for run in runs:
        if run.name == 'default':
            await api_session.runs.delete(run.id)


@click.command()
@click.option('--reset', default=False, is_flag=True,
              help="Delete default game run and recreate it from scratch")
@coro
async def command(reset):
    """
    Create and initialize Calc game.
    Create a "default" Calc run.
    Set the run phase to "Play".
    Add 1 leader ("leader") to the run
    Add 2 players ("s1", "s2") to the run.
    Add a scenario and period 1 for each player.
    """

    async with games_client as api_session:

        # Handle resetting the game
        if reset:
            if click.confirm(
                    'Are you sure you want to delete the default game run and recreate from scratch?'):
                await delete_default_run(api_session)

        # Create a Game
        game = await api_session.games.get_or_create(
            name='Calc',
            slug='calc'
        )
        echo('getting or creating game: ', game.name)

        # Create game Phases ("Play")
        play_phase = await api_session.phases.get_or_create(
            game=game.id,
            name='Play',
            order=1,
        )
        echo('getting or creating phase: ', play_phase.name)

        # Add run with 2 players ready to play
        run = await add_run(game, 'default', 2, play_phase, api_session)

        echo('Completed setting up run: id=', run.id)


async def add_run(game, run_name, user_count, phase, api_session):
    # Create or get the Run
    run = await api_session.runs.get_or_create(
        game=game.id,
        name=run_name,
    )
    echo('getting or creating run: ', run.name)

    # Set run to phase
    run.phase = phase.id
    await run.save()
    echo('setting run to phase: ', phase.name)

    fac_user = await api_session.users.get_or_create(
        password='leader',
        first_name='CALC',
        last_name='Leader',
        email='leader@calc.edu',
    )
    echo('getting or creating user: ', fac_user.email)

    fac_runuser = await api_session.runusers.get_or_create(
        user=fac_user.id,
        run=run.id,
        leader=True,
    )
    echo('getting or creating leader runuser for user: ', fac_user.email)

    for n in range(0, user_count):
        user_number = n + 1
        # Add player to run
        await add_player(user_number, run, api_session)

    return run


async def add_player(user_number, run, api_session):
    """Add player with name based on user_number to run with role"""

    username = 's{0}'.format(user_number)
    first_name = 'Student{0}'.format(user_number)
    email = '{0}@calc.edu'.format(username)

    user = await api_session.users.get_or_create(
        password=username,
        first_name=first_name,
        last_name='User',
        email=email,
    )
    echo('getting or creating user: ', user.email)

    runuser = await api_session.runusers.get_or_create(
        user=user.id,
        run=run.id,
        defaults={"role": None}
    )
    echo('getting or creating runuser for user: ', user.email)

    await add_runuser_scenario(runuser, api_session)


async def add_runuser_scenario(runuser, api_session):
    """Add a scenario named 'Scenario 1' to the runuser"""

    scenario = await api_session.scenarios.get_or_create(
        runuser=runuser.id,
        name='Scenario 1',
    )
    click.echo('getting or creating runuser {} scenario: {}'.format(
        runuser.id,
        scenario.id))

    period = await api_session.periods.get_or_create(
        scenario=scenario.id,
        order=1,
    )
    click.echo('getting or creating runuser {} period 1 for scenario: {}'.format(
        runuser.id,
        scenario.id))

```

Run your command:

```shell
$ export DJANGO_SETTINGS_MODULE=calc_model.settings
$ ./manage.py create_default_env
```

Every player's submission will be a `Decision` saved on the current `Period`. The model will then produce a `Result` for the
current `Period`, and the player's `Scenario` will step to the next `Period`.

In your `game` app module, create a file called `runmodel.py`.  Next, add `save_decision` and `step_scenario` functions to perform these steps:

```python
from modelservice.simpl import games_client
from .model import Model


async def save_decision(period_id, decision):
    # add decision to period
    async with games_client as api_session:
        decision = await api_session.decisions.get_or_create(
            period=period_id,
            name='decision',
            data={"operand": decision},
            defaults={"role": None}
        )
        return decision


async def step_scenario(scenario_id):
    """
    Step the scenario's current period
    """
    async with games_client as api_session:
        periods = await api_session.periods.filter(scenario=scenario_id,
                                                   ordering='order')
        period_count = len(periods)
        period = periods[period_count - 1]

        operand = 0.0
        period_decisions = await api_session.decisions.filter(period=period.id)
        if len(period_decisions) > 0:
            operand = float(period_decisions[0].data["operand"])

        prev_total = 0.0
        if period_count > 1:
            prev_period = periods[period_count - 2]
            prev_period_results = \
                await api_session.results.filter(period=prev_period.id)
            if len(prev_period_results) > 0:
                prev_total = float(prev_period_results[0].data["total"])

        # step model
        model = Model()
        total = model.step(operand, prev_total)
        data = {"total": total}

        result = await api_session.results.get_or_create(
            period=period.id,
            name='results',
            data=data,
            defaults={"role": None}
        )

        # prepare for next step by adding a new period
        next_period_order = period.order + 1
        next_period = await api_session.periods.get_or_create(
            scenario=scenario_id,
            order=next_period_order,
        )
        await next_period.save()

        return next_period.id
```


In your `game` app module, create a file called `games.py` with the following content:

```python
from modelservice.games import Period, Game
from modelservice.games import subscribe, register

from .runmodel import step_scenario, save_decision


class CalcPeriod(Period):
    @subscribe
    async def submit_decision(self, operand, **kwargs):
        """
        Receives the operand played and stores as a ``Decision`` then
        steps the model saving the ``Result``. A new ``Period`` is added to
        scenario in preparation for the next decision.
        """
        # Call will prefix the ROOT_TOPIC
        # "world.simpl.sims.calc.model.period.1.submit_decision"

        for k in kwargs:
            self.session.log.info("submit_decision: Key: {}".format(k))

        await save_decision(self.pk, operand)
        self.session.log.info("submit_decision: saved decision")

        await step_scenario(self.scenario.pk)
        self.session.log.info("submit_decision: stepped scenario")


Game.register('calc', [
    CalcPeriod,
])
```

**NOTE:** if you
want to use a filename other than `games.py` you must ensure the file is imported
somewhere, usually in a `__init__.py` somewhere for the `@game` decorator to find
and register your game into the system.


You can start your model service by running:

```shell
$ export DJANGO_SETTINGS_MODULE=calc_model.settings
$ ./manage.py run_modelservice
```

By default the service will bind to `0.0.0.0:8080`.

This concludes the tutorial on building a single-player game Model Service. A completed example implementation is available at 
[https://github.com/simplworld/simpl-calc-model](https://github.com/simplworld/simpl-calc-model) that uses the game slug `simpl-calc`.

You can now head over to the [Single-player Game Frontend tutorial]({% link _docs/tutorials/single-player/frontend.md %}).
