---
layout: post
title: "Load Test Your Model Service"
date: 2021-05-18 15:00:00 -0400
categories: Advanced
author: Jane Eisenstein
author_photo: /assets/img/authors/jane-eisenstein.png
featured: false
excerpt: |
    Use model service profiling to performance tune your games.
---


Work on the Simpl simulation platform began in early 2016. The Wharton Learning Lab ran its first Simpl game for a class of 300 students in the fall of 2017. 
This launch was almost a disaster due to the poor performance of the game's model service. Afterwards, the `simpl-modelservice` package was rewritten 
to use asynchronous requests and to provide performance profiling. Using `simpl-modelservice` profiling, the development team performance tuned that 
first Simpl game against 300 simulated players. This led to the game running successfully for classes of 300+ students ever since.

## The simpl-modelservice Profiling Functionality 

The `ProfileCase` class is used in the game's model service to create profile tasks:

```python
class ProfileCase(unittest.TestCase):
    """
    A class for grouping profile tasks.

    Inherits from ``unittest.TestCase`` so that we can reuse unittest's discovery.

    Unlike ``unittest.TestCase``, it does not support ``.setUp()`` or ``.tearDown()`` methods.

    A ProfileCase contains multiple ``profile_*`` tasks.
...
    """
```

The `profile` management command emulates a single user session that may send HTTP requests to the Simpl server and WAMP messages to the game's modelservice.
A test user email address is provided to the tasks defined by in game's model service's **profile_\*** ProfileCase subclass.

The `profile.sh` script asynchronously runs the `profile` command against a series of test user email addresses stored in a text file and
reports how many seconds it takes for all tasks to complete. 

## Using simpl-modelservice Profiling

To add a profile task to your model service:

* Add a `profilers` module to the `game` module.
  
* Add a file named `profile_test.py` to the `profilers` module.

* Define a profile task in `profile_test.py` whose name starts with 'profile_'.

A common profile task pattern simulates a player logging into the game and submitting decisions. 
In this pattern, the task retrieves information about the test player from the Simpl server, 
emulates the `simpl-react` **simpl** decorator's initial WAMP requests, 
then submits decisions for the test player. 
Example profile tasks with this pattern are in the Simpl `simpl-div-model` and `simpl-calc-model` repositories. 

In the multi-player **Simpl Div** game's `simpl-div-model` repository, the `profile_test.py` file contents look like:

```python
import asyncio

from django.conf import settings

from modelservice.profiler import ProfileCase
from modelservice.simpl import games_client_factory


class ProfileTestCase(ProfileCase):
    """
    Profiles HTTP calls to simpl-games-api and WAMP calls to the modelservice.
    """

    async def profile_submit_decision(self):
        email = self.user_email
        if email is not None:

            # email format is <char><int>@ where <int> is 1..78
            # which assumes run name is a single letter
            decision = int(email[1:email.find('@')])
            
            password = email[0:email.find('@')]

            coro_client = games_client_factory()

            async with coro_client as coro_session:
                try:
                    # Determine Run and Runuser based on player email
                    run_name = email[0]  # run name is a single letter
                    run = await coro_session.runs.get(
                        game_slug=settings.GAME_SLUG,
                        name=run_name,
                    )
                    user = await coro_session.users.get(email=email)
                    runuser = await coro_session.runusers.get(
                        run=run.id,
                        user=user.id
                    )

                    # From here down, pull data from modelservice via WAMP

                    # First, emulate calls made by the simpl-react simpl decorator when a player logs in

                    world_topic = 'world.simpl.sims.simpl-div.model.world.' + str(runuser.world)

                    #  getRunUsers(world_topic)
                    get_active_runusers_uri = world_topic + '.get_active_runusers'
                    get_active_runusers_result = await self.call(get_active_runusers_uri)

                    # getCurrentRunPhase(world_topic)
                    get_current_run_and_phase_uri = world_topic + '.get_current_run_and_phase'
                    get_current_run_and_phase_result = await self.call(get_current_run_and_phase_uri)

                    # getDataTree(world_topic)
                    get_scope_tree_uri = world_topic + '.get_scope_tree'
                    get_scope_tree_result = await self.call(get_scope_tree_uri)

                    runuser_topic = 'world.simpl.sims.simpl-div.model.runuser.' + str(runuser.id)

                    # getRunUserScenarios(runuser_topic)
                    get_scenarios_uri = runuser_topic + '.get_scenarios'
                    get_scenarios_result = await self.call(get_scenarios_uri)

                    # getPhases('model:model.game')
                    get_phases_uri = 'world.simpl.sims.simpl-div.model.game.get_phases'
                    get_phases_ = await self.call(get_phases_uri)

                    # getRoles('model:model.game')
                    get_roles_uri = 'world.simpl.sims.simpl-div.model.game.get_roles'
                    get_roles_result = await self.call(get_roles_uri)

                    # Next, prepare to submit player's decision

                    # Check whether run is in Play phase
                    run_phase_name = get_current_run_and_phase_result['phase']['data']['name']
                    if run_phase_name != 'Play':
                        raise Exception("ERROR: Run must be in Play phase")

                    if len(get_scope_tree_result['children'][0]['children']) > 1:
                        raise Exception("ERROR: Player's world has more than one period")

                    # Check whether this world already has a result
                    first_period = get_scope_tree_result['children'][0]['children'][0]
                    if len(first_period['children']) == 3:
                        raise Exception("ERROR: Player's world already has a result")

                    # get id of first period of first scenario of player's world
                    first_period_id = first_period['pk']

                except Exception as e:
                    print(e)
                    return

                # submit player's decision
                uri = 'world.simpl.sims.simpl-div.model.period.' + \
                      str(first_period_id) + '.submit_decision'

                if decision is not None:
                    status = await self.call_as(email, password, uri, decision)
                    if status != 'ok':
                        raise ValueError(
                            "submit_decision: status=" + status)
```

The single-player **Simpl Calc** game's `simpl-calc-model` repository's profile task is similar. 
However, it implements the series of calls the `simpl-react` **simpl** decorator makes for single-player games.

```python
import asyncio

from django.conf import settings

from modelservice.profiler import ProfileCase
from modelservice.simpl import games_client_factory


class ProfileTestCase(ProfileCase):
    """
    Profiles HTTP calls to simpl-games-api and WAMP calls to the modelservice.
    """

    async def profile_submit_decision(self):
        email = self.user_email
        if email is not None:

            # email format is <char><int>@ where <int> is 1..78
            # which assumes run name is a single letter
            decision = int(email[1:email.find('@')])
            
            password = email[0:email.find('@')]

            coro_client = games_client_factory()

            async with coro_client as coro_session:
                try:
                    # Determine Run and Runuser based on player email
                    run_name = email[0]  # run name is a single letter
                    run = await coro_session.runs.get(
                        game_slug=settings.GAME_SLUG,
                        name=run_name,
                    )
                    user = await coro_session.users.get(email=email)
                    runuser = await coro_session.runusers.get(
                        run=run.id,
                        user=user.id
                    )

                    # From here down, pull data from modelservice via WAMP

                    # First, emulate calls made by the simpl-react simpl decorator when a player logs in

                    runuser_topic = 'world.simpl.sims.simpl-calc.model.runuser.' + str(runuser.id)

                    #  getRunUsers(runuser_topic, false)
                    runusers_topic = runuser_topic + '.get_active_runusers'
                    await self.call(runusers_topic)  # ignore results

                    # getCurrentRunPhase(runuser_topic)
                    await self.call(runuser_topic + '.get_current_run_and_phase')  # ignore results

                    # getDataTree(runuser_topic)
                    get_scope_tree_uri = runuser_topic + '.get_scope_tree'
                    get_scope_tree_result = await self.call(get_scope_tree_uri)

                    # getRunUserScenarios(runuser_topic)
                    get_scenarios_uri = runuser_topic + '.get_scenarios'
                    get_scenarios_result = await self.call(get_scenarios_uri)

                    # getPhases('model:model.game')
                    get_phases_uri = 'world.simpl.sims.simpl-calc.model.game.get_phases'
                    get_phases_ = await self.call(get_phases_uri)

                    # getRoles('model:model.game')
                    get_roles_uri = 'world.simpl.sims.simpl-calc.model.game.get_roles'
                    get_roles_result = await self.call(get_roles_uri)

                    # Next, prepare to submit player's decision

                    # as scope tree children are not returned in order, avoid sorting periods
                    if len(get_scope_tree_result['children'][0]['children']) > 1:
                        raise Exception("ERROR: Player's scenario has more than one period")

                    # get id of last period of player's scenario, which is also its first period
                    periods = get_scope_tree_result['children'][0]['children']
                    last_period_id = periods[len(periods) - 1]['pk']

                except Exception as e:
                    print(e)
                    return

                # submit player's decision against the last period
                uri = 'world.simpl.sims.simpl-calc.model.period.' + str(last_period_id) + '.submit_decision'

                status = await self.call_as(email, password, uri, decision)
                if status != 'ok':
                    raise ValueError(
                        "submit_decision: status=" + status)
```

Before running a profile task, you also need to:

* Add a **GAME_SLUG** setting that matches your game's slug (i.e. GAME_SLUG = ‘simpl-calc’).

* Create one or more test game runs ready to be played.

* Create a text file containing the email addresses of players in the test game runs.

The Simpl `simpl-div-model` and `simpl-calc-model` repositories master branches illustrate how this is done.
The README files in both repositories contain instructions for creating test game runs and running profiling locally.

## Performance tuning your deployed game

Once you have profiling running locally, you are ready to get real life timings against your deployed game.

TODO

## Summary

Using `simpl-modelservice` profiling allows developers to gauge throughput of their game under load and identify bottle necks that need correction. 
Performance tuning your game before its first launch contributes to peace of mind.






