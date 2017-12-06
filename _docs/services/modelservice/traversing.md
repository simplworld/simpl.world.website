# Traversing Scopes

Every Scope has a `.my` attribute that can be user to reach other scopes that are related to a specific instance.

For example, to get the parent scope of `myscope`, you can use `scope.my.parent`.

From any Scope, you can reach the following scope:

* `.game`. The instance of the root (ie: top-level) scope in the chain. Think of this as the 'global' state for this game.
* `.run`. The instance of the `run` scope in the chain. This will be only set for scopes downstream from any run.
* `.world`. The instance of the `worlds scope in the chain. This will be only set for scopes downstream from any world.
* `.parent`. The parent scope. This represent 'one level above' in the scope chain.

Additionally, the following methods are available:

* `.get_runusers(leader=False)`: Returns a [`ScopeManager`](./scopemanager.md) of `RunUser`s that have access to the scope. If `leader` is True and the scope is a world, returns all RunUsers for the World's Run
* `.get_users(leader=False)`: returns a list of user ids (not runuser ids) from `get_runusers`.
