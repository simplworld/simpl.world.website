# ScopeManager

Collection of scopes are usually returned as ScopeManager.

A `ScopeManager` behaves like a list (ie: you can iterate on them, use the `in` operator), and allows you to operate on collections of scopes.

## Filtering

`ScopeManager.filter(**kwargs)` will return a new `ScopeManager` with only the scopes that match the conditions.

Example:

```python
runusers.filter(world=3)  # returns only scopes that have `.json['world']` set to `3`.
```

`ScopeManager.resource_name(resource_name)` will return only the scopes of type `resource_name`.

Example:

```python
period.child_scopes.resource_name('result')
```

`ScopeManager.for_user(user)` will return only the subset of scopes that a specific user (not to be confused with `RunUser`) as access to through their `RunUser`s.

Similar to `.filter`, `.get(**kwargs)` will retrieve only one scope matching the conditions.

If no one scope cane found, `ScopeManager.ScopeNotFound` will be raised.

If more than one scope is found, `ScopeManager.MultipleScopesFound` will be raised.

Example:

```python
myscopes.get(world=3)  # return a scopes that have `.json['world']` set to `3`.
```

You can get the last scope of a manager with the `.last()` method.

`.count()` will return the number of scopes in the manager.

## Editing

Just like list, `ScopeManagers` can be combined using the `+` operator:

```python
my_scopes = manager1 + manager2
```

They also provide an `append(*scopes)` method:

```python
my_scopes = manager1.append(scope1, scope2)
```

Finally, you can clear out a manager with `.reset()`.
