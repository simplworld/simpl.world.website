---
title: The Simpl Client
permalink: /docs/services/modelservice/simpl/
layout: docs
description:
---

## The Simpl Client

The `modelservice.simpl.games_client` class provides a generic asynchronous REST API client that you can use to
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

### Resource methods

* `.save()`
* `.delete()`

### Resource attributes

* `.payload`: a `dict` containg the data returned in the response body.


### Exceptions

All exceptions are subclasses of `ValueError`.

* `MultipleResourcesFound`
* `ResourceNotFound`
* `NotAuthenticatedError`
* `HTTPError`
