---
title: Games Client
permalink: /docs/services/modelservice/games-client/
layout: docs
description:
---

## Games Client

The `modelservice.simpl.games_client` provides a generic asynchronous REST API client that you can use to
interface with the Simpl-Games-API service.


```python
from modelservice.simpl import games_client

    async with games_client as api_session:
    
        users = await api_session.users.all()
    
        world_users = await api_session.users.filter(world=35)
    
        user = await api_session.users.get(email='myuser@example.com')
    
        user.first_name = 'Jessie'
    
        await user.save()
```

### Endpoint methods

* `.create_or_update(**kwargs)`
* `.create(**kwargs)`
* `.all()`
* `.filter(**kwargs)`
* `.get(**kwargs)`

### Bulk Endpoint methods

* `.create([...], return_ids=False)`
* `.delete()`

### Resource methods

* `.save()`
* `.delete()`

### Resource attributes

* `.payload`: a `dict` containing the data returned in the response body.

The payload's keys can also be accessed as properties of the Resource.

### Detail Routes

```python
    await games_client.scenario(id=123).rewind() 
```

### Exceptions

All exceptions are subclasses of `ValueError`.

* `MultipleResourcesFound`
* `ResourceNotFound`
* `NotAuthenticatedError`
* `HTTPError`


## Cooperative Multi-tasking Simpl Data Access

In conjunction with the Simpl-Modelservice `spawn` function, multiple Games Client instances can be 
used to implement cooperative multi-tasking.

```python

from modelservice.simpl import games_client_factory
from modelservice.utils.asyncio import spawn

async def rewind_world_scenarios(run_id: int, api_session):
    """
    Resets all run's world scenarios to period 1 maintaining current decisions
    and deleting all results
    """

    # Get run's world scenario ids
    scenario_ids = [scenario.id for scenario in
                    await api_session.scenarios.filter(world__run=run_id)]

    await spawn(rewind_scenario, scenario_ids)


async def rewind_scenario(scenario_id):
    """
    Resets scenario to period 1 maintaining current decisions
    and deleting all results
    """
    coro_client = games_client_factory()

    async with coro_client as coro_session:
        await coro_session.scenarios(id=scenario_id).rewind(
            last_period_order=1,
            delete_last_period_decisions=False
        )
```