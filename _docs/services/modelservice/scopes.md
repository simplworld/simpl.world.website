---
title: Games
permalink: /docs/services/modelservice/games/
layout: docs
description:
---

## Games

### Scopes

Every game is defined by building what we call the _Scope tree_.

The _Scope tree_ starts at `Game`, and it's defined as follow:

* `Game`
    * `Phase`
    * `Role`
    * `Run`
        * `RunUser`
        * `World`
            * `Scenario`
                * `Period`
                    * `Decision`
                    * `Result`

The `modelservice.games` module provides all the base classes to build this tree. Each of this classes is considered a _Scope_. The logic of your game will be implemented by subclassing the necessary scope and adding your own custom logic.

Your game is defined by registering your top-level Scope with the `modelservices.games.game` decorator.

Inside any Scope, you can make any method callable from javascript by decorating it with the `modelservice.games.register` decorator.
Similarly, you can use the `modelservice.games.subscribe` decorator to subscribe any method to a topic:

```python
from modelservice.games import Scenario, Game
from modelservice.games import register, subscribe


class ZeroSumScenario(Scenario):
    players = {}

    @register
    def add(self, term1, term2, details):
        return term1 + term2

    @subscribe
    def player_quit(self, player, reason, details):
        self.players.pop(player['id'])

Game.register('zero-sum', [ZeroSumScenario])
```

In this example, `ZeroSumScenario.add` will be registered at the uri `{settings.ROOT_URI}.models.{resource_name}.{scope_pk}.add`
 and `ZeroSumScenario.player_quit` will subscribe to `{settings.ROOT_URI}.models.resource_name}.{scope_pk}.player_quit`.

Alternatively, the name that will be used for the topic can be passed to the decorator:

```python
class ZeroSumScenario(Scenario):

    @subscribe('player.quit')
    def player_quit(self, player, reason):
        self.players.pop(player['id'])

Game.register('zero-sum', [ZeroSumScenario])
```

`ZeroSumScenario.player_quit` will now be subscribed to `{settings.ROOT_URI}.models.{resource_name}.{scope_pk}.player.quit`.

Further customization can be achieved by overriding the `Scope.get_routing(name)` method.

#### Scope methods

All _Scopes_ inherit from a common `Scope` subclass and share these common methods among many others:

* `.get_routing(name)`: given a topic short name, returns the full topic path. This is to avoid repeating the same topic namespace over and over, and allow using shorter topic names.
* `.publish(topic, *args, **kwargs)`: Publishes a message on the specified `topic`. The actual topic name will be what is returned by `.get_routing(topic)`
* `.save()`: Persist the scope by submitting it to the `Simpl-Games-API` service.
* `.get_scope()`: Registered procedure at `{scope.topic}.get_scope` returning the scope's serialized version. Call this procedure if you want to retrieve the scope's state from the UI
* `.get_scope_tree()`: Same as `.get_scope`, but will also walk down the tree from scope, returning its children recursively.
* `.onConnected()`
* `.onStop()`
* `.onStart()`

#### Scope properties

All _Scopes_ inherits from a common `Scope` subclass and share these common properties among many others:

* `.pk`: The scopes primary key
* `.json`: A `dict` that contains the serialized representation of the scope. This is what will be passed to the browser
and to the Simpl-Games-API service. Any user-defined state for the scope should be stored in `.json['data']`
* `.game`: The `Game` with which this scope belongs.
* `.child_scopes`: A [`ScopeManager`](./scopemanager.md) of the children scope.
* `.games_client`: An instance of a REST API client for the Simpl-Games service.
* `.my`: This special property will contain scopes that are related to the scope. See [Traversing Scopes](./traversing.md) for more details.
* `.log`: a `txaio` logging instance. See "Logging" below.

#### Logging

To log from a `Scope`, you can use the `.log` attribute.

The best practice is to use a string with the `r` formatting operator for your variables. For example, if you'd want to log the `somevar` variable to the `INFO` level, you'd use:

```python
myscope.log.info("Something happened: {myvar!r}", myvar=somevar)
```

#### Concrete Classes

For convenience, the following scopes are defined in `modelservices.games`:

* `modelservices.games.Game`
* `modelservices.games.Phase`
* `modelservices.games.Role`
* `modelservices.games.Run`
* `modelservices.games.World`
* `modelservices.games.RunUser`
* `modelservices.games.Scenario`
* `modelservices.games.Period`
* `modelservices.games.Decision`
* `modelservices.games.Result`

##### modelservices.games.Game

In addition to the methods inherited from `Scope`, `Game` has the following methods:

* `.get_phases()`: Registered procedure to fetch the game's phases over WAMP.
* `.get_roles()`: Registered procedure to fetch the game's roles over WAMP.

In addition to the properties inherited from `Scope`, `Game` has the following properties:

* `.phases`: A list of the game's `Phase`s.
* `.roles`: A list of the game's `Roles`s.
* `.runs`: A list of the game's `Run`s.

##### modelservices.games.Run

In addition to the methods inherited from `Scope`, `Run` has the following methods:

* `.advance_phase()`: Subscribed method to make the `Run` advance to the next phase.
* `.on_advance_phase(next_phase)`: Override this method to perform custom logic right after the run is advanced to the next phase.
* `.rollback_phase()`: Subscribed method to make the `Run` rollback to the previous phase.
* `.on_rollback_phase(previous_phase)`: Override this method to perform custom logic right after the run is rolled back to the previous phase.

In addition to the properties inherited from `Scope`, `Run` has the following properties:

* `.current_phase`: The Run's current phase.
* `.worlds`: A list of `World`s within this `Run`
* `.runusers`: A list of `RunUsers`s within this `Run`

##### modelservices.games.World

In addition to the properties inherited from `Scope`, `World` has the following properties:

* `.run`: The Run to which this World belongs.
* `.scenarios`: A list of the World's `Scenarios`s.
* `.runusers`: A list of the World's `RunUser`s.

##### modelservices.games.RunUser

In addition to the methods inherited from `Scope`, `RunUser` has the following methods:

* `.get_scenarios()`: Registered procedure to fetch the RunUser's Scenarios over WAMP.

In addition to the properties inherited from `Scope`, `Run` has the following properties:

* `.run`: The Run to which this Runuser belongs.
* `.scenarios`: A list of the RunUser's `Scenarios`s.
* `.leader`: Flags whether the RunUser is a leader or a player in this `Run`.
* `.role`: The RunUser's `Role` if the user is a player in this `Run`.
* `.world`: The RunUser's `World` if the user is a player in this `Run` and belongs to a `World`.

##### modelservices.games.Scenario

In addition to the properties inherited from `Scope`, `Scenario` has the following properties:

* `.periods`: A list of the Scenario's `Period`'s.
* `.world`: returns the `World` to which the Scenario belongs or `None` if the Scenario doesn't belong to a `World`.
* `.runuser`: returns the `Runuser` to which the Scenario belongs or `None` if the Scenario doesn't belong to a `Runuser`.

##### modelservices.games.Period

In addition to the properties inherited from `Scope`, `Period` has the following properties:

* `.decisions`: A list of the Period's `Decisions`s.
* `.results`: A list of the Period's `Result`s.
* `.scenario`: returns the `Scenario` to which the Period belongs.

##### modelservices.games.Decision

In addition to the properties inherited from `Scope`, `Decision` has the following properties:

* `.period`: returns the `Period` to which the Decision belongs.
* `.role`: returns the Decision's `Role`.

##### modelservices.games.Result

In addition to the properties inherited from `Scope`, `Result` has the following properties:

* `.period`: returns the `Period` to which the Result belongs.
* `.role`: returns the Result's `Role`.
