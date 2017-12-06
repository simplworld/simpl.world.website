# The simpl client

The `modelservice.simpl.games_client` class provides a generic REST API client that you can use to interface with the Simpl-Games service.


```
from modelservice.simpl import games_client

users = games_client.users.all()

world_users = games_client.users.filter(world=35)

user = games_client.users.get(email='myuser@example.com')

user.first_name = 'Jessie'

user.save()

```

## Endpoint methods

* `.create_or_update(**kwargs)`
* `.create(**kwargs)`
* `.all()`
* `.filter(**kwargs)`
* `.get(**kwargs)`

## Resource methods

* `.save()`
* `.delete()`

## Resource attributes

* `.payload`: a `dict` containg the data returned in the response body.


## Exceptions

All exceptions are subclasses of `ValueError`.

* `MultipleResourcesFound`
* `ResourceNotFound`
* `NotAuthenticatedError`
* `HTTPError`
