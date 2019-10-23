---
title: Build the Multi-player Game Model Service
permalink: /docs/tutorials/multi-player/modelservice/
layout: docs
description:
---

## Build the Multi-player Game Model Service

###  Prerequisites

You will need to have these installed:
   * PostgreSQL >= 9.6
   * Python == 3.6
   * [virtualenv](https://virtualenv.pypa.io/en/stable/)

Have the [Games API service]({% link _docs/getting-started.md %}) is running on http://localhost:8100/.  

###  Installation

In a separate terminal, create a new virtualenv called 'div-model':

```shell
$ mkvirtualenv div-model
```

Install Django

```shell
$ pip install Django~=2.2.0
```

Create a Django project folder and rename it to serve as a git repository

```shell
$ django-admin startproject div_model
$ mv div_model div-model
```

Change to the project folder:

```shell
$ cd div-model
$ add2virtualenv .
```

Create a `requirements.txt` file that installs the simpl-modelservice and unit testing apps:

```ini
git+https://github.com/simplworld/simpl-modelservice.git

# tests
pytest==4.6.3
pytest-cov==2.7.1
pytest-django==3.5.0
django-test-plus==1.3.1
```

Install these requirements along with their dependencies:

```shell
$ pip install -r requirements.txt
```

Note, if you receive any errors when installing the above requirements, make sure you have python3.6-dev installed.

Please note, if `DJANGO_SETTINGS_MODULE` is leftover from a previous session, you may need to unset it:

```shell
$ unset DJANGO_SETTINGS_MODULE
```

Create a django app that will contain your game logic:

```shell
$ ./manage.py startapp game
```

Add the following to your `INSTALLED_APPS` in `div_model/settings.py`:

```python
INSTALLED_APPS += [
    'modelservice',
    'rest_framework',
    'game',
]

CALLBACK_URL = os.environ.get('CALLBACK_URL', 'http://{hostname}:{port}/callback')

SIMPL_GAMES_URL = os.environ.get('SIMPL_GAMES_URL', 'http://localhost:8100/apis')

SIMPL_GAMES_AUTH = ('simpl@simpl.world', 'simpl')

ROOT_TOPIC = 'world.simpl.sims.div'
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

For simplicity, we're going to create a multi-player Game in which each world must have exactly two players -- 
one playing the Dividend role, the other playing the Divisor role.
The game's model will automatically advance as soon as both players in a world have submitted valid decisions for their roles.

In your `game` app module, define the simulation model by creating a `model.py` file with the following content:

```python
class Model(object):
    """
    The model calculates a result given a dividend and a divisor and returns the result.
    """

    def step(self, dividend, divisor):
        """
        Parameters:
            dividend - current period's dividend decision
            divisor - current period's divisor decision
        Returns result of dividing dividend by divisor.
        """
        return dividend / divisor
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

    def test_result_5(self):
        result = self.m.step(1.25, 0.25)
        self.assertEqual(result, 5)

    def test_result_fraction(self):
        result = self.m.step(1, 2)
        self.assertEqual(result, 0.5)
```

Run your unit test:

```shell
$ export DJANGO_SETTINGS_MODULE=div_model.settings
$ py.test
```

Create a management command that will create your game and initialize it with one run, a leader, 2 worlds, and 2 players per world.

Create a `management` folder in the `game` folder and add an empty `__init__.py` file.

Create a `commands` folder in the `game/management` folder  and add an empty `__init__.py` file.

Finally, create a `create_default_env.py` script in the `game/management/commands` folder containing this code:

```python
import djclick as click

from modelservice.simpl.sync import games_client


def echo(text, value):
    click.echo(
        click.style(text, fg='green') + '{0}'.format(value)
    )


def delete_default_run(games_client):
    """ Delete default Run """
    echo('Resetting the Div game default run...', ' done')
    runs = games_client.runs.filter(game_slug='div')
    for run in runs:
        if run.name == 'default':
            games_client.runs.delete(run.id)


@click.command()
@click.option('--reset', default=False, is_flag=True,
              help="Delete default game run and recreate it from scratch")
def command(reset):
    """
    Create and initialize Div game.
    Create a "default" Div run.
    Set the run phase to "Play".
    Add 2 worlds to the run.
    Add a scenario and period 1 for each world.
    Add 1 leader ("leader") to the run
    Add 4 players ("s1", "s2", "s3", "s4") to the run
    splitting the players between the 2 worlds and assigning all roles.
    """

    # Create a Game
    game = games_client.games.get_or_create(
        name='Div',
        slug='div'
    )
    echo('getting or creating game: ', game.name)

    # Handle resetting the game
    if reset:
        if click.confirm(
                'Are you sure you want to delete the default game run and recreate from scratch?'):
            delete_default_run(games_client)

    # Create required Roles ("Dividend" and "Divisor")
    dividend_role = games_client.roles.get_or_create(
        game=game.id,
        name='Dividend',
    )
    echo('getting or creating role: ', dividend_role.name)

    divisor_role = games_client.roles.get_or_create(
        game=game.id,
        name='Divisor',
    )
    echo('getting or creating role: ', divisor_role.name)

    # Create game Phases ("Play" and "Debrief")
    play_phase = games_client.phases.get_or_create(
        game=game.id,
        name='Play',
        order=1,
    )
    echo('getting or creating phase: ', play_phase.name)

    debrief_phase = games_client.phases.get_or_create(
        game=game.id,
        name='Debrief',
        order=2,
    )
    echo('getting or creating phase: ', debrief_phase.name)

    # Add run with 2 fully populated worlds ready to play
    run = add_run(game, 'default', 2, 1,
                  dividend_role, divisor_role,
                  play_phase, games_client)

    # echo('Completed setting up run: id=', run.id)


def add_run(game, run_name, world_count, first_user_number,
            dividend_role, divisor_role,
            phase, games_client):
    # Create or get a Run
    run = games_client.runs.get_or_create(
        game=game.id,
        name=run_name,
    )
    echo('getting or creating run: ', run.name)

    # Set run to phase
    run.phase = phase.id
    run.save()
    echo('setting run to phase: ', phase.name)

    user_name_root = "s"
    if run_name is not 'default':
        user_name_root = run_name
    for n in range(0, world_count):
        world_num = n + 1
        world = add_world(run, world_num, games_client)

        # Add users to run
        add_world_users(run, world, user_name_root,
                        first_user_number + n * 2,
                        dividend_role, divisor_role, games_client)

    return run


def add_world(run, number, games_client):
    """
        Add a world to the run with a scenario and period 1.
        The world's name is based on number.
    """
    name = 'World {0}'.format(number)
    world = games_client.worlds.get_or_create(
        run=run.id,
        name=name,
    )
    echo('getting or creating world: ', world.name)

    scenario = games_client.scenarios.create({
        'world': world.id,
        'name': 'World Scenario 1'
    })
    period1 = games_client.periods.create({
        'scenario': scenario.id,
        'order': 1
    })

    return world


def add_world_users(run, world, user_name_root,
                    first_number,
                    dividend_role, divisor_role, games_client):
    """
        Add 1 leader ("leader") to the run with a test scenario
        Add players to the run with names based on user_name_root and first_number
        Add players to world assigning all required roles
    """
    fac_user = games_client.users.get_or_create(
        password='leader',
        first_name='Div',
        last_name='Leader',
        email='leader@div.edu',
    )
    echo('getting or creating user: ', fac_user.email)

    games_client.runusers.get_or_create(
        user=fac_user.id,
        run=run.id,
        leader=True,
    )
    echo('getting or creating leader runuser for user: ', fac_user.email)

    roles = [dividend_role, divisor_role]
    for n in range(len(roles)):
        user_number = n + first_number
        add_player(user_name_root, user_number, run, world, roles[n],
                   games_client)


def add_player(user_name_root, user_number, run, world, role,
               games_client):
    """Add player with name based on user_name_root and user_number to world in role"""

    username = '{}{}'.format(user_name_root, user_number)
    first_name = 'Student{0}'.format(user_number)
    if user_name_root == 's':  # assume original default namings
        last_name = 'User'
    else:
        last_name = user_name_root[:1].upper() + user_name_root[1:]
    email = '{0}@div.edu'.format(username)

    user = games_client.users.get_or_create(
        password=username,
        first_name=first_name,
        last_name=last_name,
        email=email,
    )
    echo('getting or creating user: ', user.email)

    games_client.runusers.get_or_create(
        user=user.id,
        run=run.id,
        world=world.id,
        role=role.id,
    )
    echo('getting or creating runuser for user: ', user.email)
```

Run your command:

```shell
$ export DJANGO_SETTINGS_MODULE=div_model.settings
$ ./manage.py create_default_env
```

A `World`'s players will each submit a `Decision` saved on the `World` `Scenario`'s current `Period`. After both players have submitted a valid decision,
the model will  produce a `Result` for the current `Period`, and the `World`'s `Scenario` will step to the next `Period`.

In your `game` app module, create a file called `runmodel.py`.  Next, add `save_decision` and `divide` functions to perform these steps:

```python
from modelservice.simpl import games_client
from .model import Model


async def save_decision(period_id, role_id, operand):
    # add/update role's decision for period
    async with games_client as api_session:
        decision = await api_session.decisions.get_or_create(
            period=period_id,
            name='decision',
            role=role_id
        )
        decision.data["operand"] = float(operand)
        await decision.save()
        return decision


async def divide(period_id):
    """
    (Re)calculates the result of the period's Dividend and Divisor decisions.
    """
    async with games_client as api_session:
        period = await api_session.periods.get(scenario=period_id)
        period_decisions = await api_session.decisions.filter(period=period.id)

        dividend, divisor = None, None
        for decision in period_decisions:
            role = await api_session.roles.get(id=decision.role)
            if role.name == 'Dividend':
                dividend = decision.data["operand"]
            else:
                divisor = decision.data["operand"]

        if dividend is None or divisor is None:
            return None

        # run model
        model = Model()
        quotient = model.step(dividend, divisor)

        result = await api_session.results.get_or_create(
            period=period.id,
            name="result",
            defaults={"role": None}
        )
        result.data["quotient"] = quotient
        await result.save()

        return quotient
```


In your `game` app module, create a file called `games.py` that defines a `DivPeriod` Scope subclass with a `submit_decision` RPC method. 
The method validates the `operand` argument and returns and error message if a `Divisor` player submits a zero. 
Otherwise, the model is run if both players in a world have submitted a decision for this period:

```python
import asyncio

from modelservice.games import Period, Game
from modelservice.games import subscribe, register

from .runmodel import divide, save_decision


class DivPeriod(Period):
    @register
    async def submit_decision(self, operand, **kwargs):
        """
        Receives the operand played and saves as a ``Decision``.
        If decisions for both roles have been saved,
        runs the model saving the ``Result``.
        """
        # Call will prefix the ROOT_TOPIC
        # "world.simpl.sims.div.model.period.1.submit_decision"

        for k in kwargs:
            self.session.log.info("submit_decision: Key: {}".format(k))

        user = kwargs['user']
        runuser = self.game.get_scope('runuser', user.runuser.pk)
        role = runuser.role

        role_name = role.json["name"]
        if role_name == "Divisor" and float(operand) == 0:
            return "Cannot divide by zero"

        await save_decision(self.pk, role.pk, operand)
        self.session.log.info(
            "submit_decision: saved decision for role {}".format(role_name))

        #pause while the scopes update
        await asyncio.sleep(0.01)

        if len(self.decisions) == 2:
            await divide(self.scenario.pk, )
            self.session.log.info("submit_decision: saved result")

        return 'ok'


Game.register('div', [
    DivPeriod,
])
```

**NOTE:** if you
want to use a filename other than `games.py` you must ensure the file is imported
somewhere, usually in a `__init__.py` somewhere for the `@game` decorator to find
and register your game into the system.


You can start your model service by running:

```shell
$ export DJANGO_SETTINGS_MODULE=div_model.settings
$ ./manage.py run_modelservice
```

By default the service will bind to `0.0.0.0:8080`.

This concludes the tutorial on building a multi-player game Model Service. A completed example implementation is available at 
[https://github.com/simplworld/simpl-div-model](https://github.com/simplworld/simpl-div-model) that uses the game slug `simpl-div`.

You can now head over to the [Multi-player Game Frontend tutorial]({% link _docs/tutorials/multi-player/frontend.md %}).
