# Games


## Scopes

Every game is defined by building what we call the _Scope tree_.

The _Scope tree_ starts at `Game`, and it's defined as follow:

* `Game`
    * `Run`
        * `RunUser`
        * `World`
            * `Scenario`
                * `Period`
                    * `Decision`
                    * `Result`

The `modelservice.games` module provides all the base classes to build this tree. Each of this classes is considered a _Scope_. The logic of your game will be implemented by subclassing the necessary scope and adding your own custom logic.

Your game is defined by registering your top-level Scope with the `modelservices.games.game` decorator.

Inside any Scope, you can make any method callable from javascript by decorating it with the `modelservice.games.register` decorator. Similarly, you can use the `modelservice.games.subscribe` decorator to subscribe any method to a topic:

```python
from modelservice.games import Scope, Game
from modelservice.games import register, subscribe


class Scenario(Scope):
    resource_name = 'scenario'
    players = {}

    @register
    def add(self, term1, term2, details):
        return term1 + term2

    @subscribe
    def player_quit(self, player, reason, details):
        self.players.pop(player['id'])

Game.register('zero-sum', [Scenario])

```

In this example, `Scenario.add` will be registered by default at the uri `{settings.ROOT_URI}.models.{resource_name}.{scope_pk}.add`, and `Scenario.player_quit` will subscribe to `{settings.ROOT_URI}.models.resource_name}.{scope_pk}.player_quit`.

The name that will be used for the function can be passed to the decorator:

```python

class Scenario(Scope):
    resource_name = 'scenario'

    @subscribe('player.quit')
    def player_quit(self, player, reason):
        self.players.pop(player['id'])

Game.register('zero-sum', [Scenario])

```

`Scenario.player_quit` will now be subscribed to `{settings.ROOT_URI}.models.{resource_name}.{scope_pk}.player.quit`.

Further customization can be achieved by overriding the `Scope.get_routing(name)` method.

### Scope methods

All _Scopes_ inherits from a common `Scope` subclass and share these common methods:

* `.get_routing(name)`: given a topic short name, returns the full topic path. This is to avoid repeating the same topic namespace over and over, and to allow to use shorter topic names.
* `.publish(topic, *args, **kwargs)`: Publishes a message on the specified `topic`. The actual topic name will be what is returned by `.get_routing(topic)` 
* `.save()`: Persist the scope by submitting it to the `Simpl-Games-API` service.
* `.add_new_child_scope(json)`
* `.get_role(role_id)`: Give a role id, returns a full-fledged `Resource` object representing the role.
* `.get_scope()`: Registered procedure at `{scope.topic}.get_scope` returning the scope's serialized version. Call this procedure if you want to retrieve the scope's state from the UI
* `.get_scope_tree()`: Same as `.get_scope`, but will also walk down the tree from scope, returning its children recursively.
* `.onConnected()`
* `.onStop()`
* `.onStart()`

### Scope properties

All _Scopes_ inherits from a common `Scope` subclass and share these common properties:

* `.pk`
* `.json`: A `dict` that contains the serialized representation of the scope. This is what will be passed to the browser and to the Simpl-Games-API service. Any user-defined state for the scope should be stored in `.json['data']`
* `.child_scopes`: A [`ScopeManager`](./scopemanager.md) of the children scope.
* `.games_client`: An instance of a REST API client for the Simpl-Games service.
* `.my`: This special property will contain scopes that are related to the scope. See [Traversing Scopes](./traversing.md) for more details.
* `.log`: a `txaio` logging instance. See "Logging" below.
* `.runuser_class`: Scope class to use for RunUsers. Override this to provide your [Custom RunUser](./custom_runuser.md). Defaults to `modelservices.games.RunUser`. Note: this only aplies to your top-level scope.

### Logging

To log from a `Scope`, you can use the `.log` attribute.

The best practice is to use a string with the `r` formatting operator for your variables. For example, if you'd want to log the `somevar` variable to the `INFO` level, you'd use:

    myscope.log.info("Something happened: {myvar!r}", myvar=somevar)

### Concrete Classes

For convenience, the following scopes are already defined in `modelservices.games`:

* `modelservices.games.Game`
* `modelservices.games.Run`
* `modelservices.games.World`
* `modelservices.games.RunUser`
* `modelservices.games.Scenario`
* `modelservices.games.Period`
* `modelservices.games.Decision`
* `modelservices.games.Result`

#### modelservices.games.Game

In addition to the methods inherited from `Scope`, `Game` has the following methods:

* `.get_phases()`: Registered procedure to fetch the game's phases over WAMP.

In addition to the properties inherited from `Scope`, `Game` has the following properties:

* `.phases`: A list of the game's `Phase`s.

#### modelservices.games.Run

In addition to the methods inherited from `Scope`, `Run` has the following methods:

* `.advance_phase()`: Subscribed method to make the `Run` advance to the next phase.
* `.on_advance_phase(next_phase)`: Override this method to perform custom logic right before the run is advanced to the next phase. Raise `Run.ChangePhaseException` to prevent the run from being advanced.
* `.rollback_phase()`: Subscribed method to make the `Run` rollback to the previous phase.

* `.on_rollback_phase(previous_phase)`: Override this method to perform custom logic right before the run is rolled back to the previous phase. Raise `Run.ChangePhaseException` to prevent the run from being rolled back.

In addition to the properties inherited from `Scope`, `Run` has the following properties:

* `.current_phase`: The current Run's phase.
* `.worlds`: A list of `World`s within this `Run`
* `.ChangePhaseException`: Exception subclass to prevent the run from being changed.

#### modelservices.games.RunUser

In addition to the methods inherited from `Scope`, `RunUser` has the following methods:

* `.get_scenarios()`: Registered procedure to fetch the RunUser's Scenarios over WAMP.
* `.add_scenario(name, payload)`: Subscriber that will add a new Scenario for the RunUser.

In addition to the properties inherited from `Scope`, `Run` has the following properties:

* `.scenarios`: A list of the RunUser's `Scenarios`s.

#### modelservices.games.Scenario

In addition to the methods inherited from `Scope`, `Scenario` has the following methods:

* `.advance()`: Stops the current period, and adds a new one.
* `.add_new_period(json)`: alias of `.add_new_child_scope`

In addition to the properties inherited from `Scope`, `Scenario` has the following properties:

* `.periods`: alias of `.child_scopes.all()`
* `.current_period`: returns the current period, defined as the last one created. Returns `None` if the scenario doesn't have any periods, yet.
* `.world`: alias of `.my.parent`

#### modelservices.games.Period

In addition to the methods inherited from `Scope`, `Period` has the following methods:

* `.add_new_decision(json)`
* `.add_new_result(json)`

In addition to the properties inherited from `Scope`, `Period` has the following properties:

* `.decisions`
* `.results`
* `.scenario`: alias of `.my.parent`

#### modelservices.games.Decision

In addition to the properties inherited from `Scope`, `Decision` has the following properties:

* `.period`: alias of `.my.parent`

#### modelservices.games.Result

In addition to the properties inherited from `Scope`, `Result` has the following properties:

* `.period`: alias of `.my.parent`
