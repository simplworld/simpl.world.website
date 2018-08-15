---
title: Settings
permalink: /docs/services/modelservice/settings/
layout: docs
description:
---

## Settings

### Required settings

```python
SIMPL_GAMES_AUTH = ('simpl@simpl.world', 'simpl')
SIMPL_GAMES_URL = os.environ.get('SIMPL_GAMES_URL', 'http://localhost:8100/apis')
CALLBACK_URL = os.environ.get('CALLBACK_URL', 'http://{hostname}:{port}/callback')
GAME_SLUG = 'game1'
ROOT_TOPIC = os.environ.get('ROOT_TOPIC', 'com.example.game1')
```

The `SIMPL_GAMES_AUTH` setting provides the email and password of a Simpl superuser. 
Run the simpl-games-api *create_simpl_user* management command to create a *simpl@simpl.world* superuser for local development.

The `SIMPL_GAMES_URL` setting provides the URL of the game's `Simpl-Games-API` service.

The `CALLBACK_URL` setting provides the URL on which the game's modelservice will be listening 
for webhook notifications from the game's `Simpl-Games-API` service.

The `GAME_SLUG` setting provides the slug value of the modelservice's game.

The `ROOT_TOPIC` setting provides a unique prefix for the game's WAMP topics.

### Optional settings

```python
LOAD_ACTIVE_RUNS = getattr(settings, 'LOAD_ACTIVE_RUNS', True)

```

The `LOAD_ACTIVE_RUNS` setting controls whether only active runs are loaded as Scopes by the modelservice. 
By default, the modelservice loads only active runs as Scopes. If set to False, both active and inactive runs will be loaded. 



