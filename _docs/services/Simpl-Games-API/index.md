---
title: Simpl-Games-API Overview
permalink: /docs/services/simple-games-api/
layout: docs
description:
---

## Simpl-Games-Api Overview

The `simpl-games-api` service provides the main storage for games' data.

In practice, there will always be one instance of The `Simpl-Games-API` service that will store data for one or more model services. Every time data is created, modified, or deleted, the `Simpl-Games-API` service will send events to the model services via HTTP Webhooks.

### Authentication

`simpl-games-api` currently supports Basic and Session authentication. You will have to create one or more users for your modelservice(s) to use.

### Endpoints

All endpoints are under the `/apis/` path.

#### REST endpoints

The following REST endpoints are available:

* `/apis/decisions/`
* `/apis/games/`
* `/apis/periods/`
* `/apis/phases/`
* `/apis/results/`
* `/apis/roles/`
* `/apis/runs/`
* `/apis/runusers/`
* `/apis/scenarios/`
* `/apis/worlds/`
* `/apis/users/`
* `/apis/hooks/`

For more info on the specific endpoint, swagger documentation is available at the root of `simpl-games-api` (ie, if you have it running on port 8100, it will be at `http://localhost:8100/`)

#### Bulk Endpoints

In addition to the RESTful endpoints, `simpl-games-api` offers endpoint to operate on multiple resources at once.

Currently, the only operation supported on these endpoints are creation (via `POST`) and deletion (via `DELETE`)

The following bulk endpoints are available:

* `/apis/bulk/decisions/`
* `/apis/bulk/games/`
* `/apis/bulk/periods/`
* `/apis/bulk/phases/`
* `/apis/bulk/results/`
* `/apis/bulk/roles/`
* `/apis/bulk/runs/`
* `/apis/bulk/runusers/`
* `/apis/bulk/scenarios/`
* `/apis/bulk/worlds/`

#### Detail Routes

As a performance optimization, the ScenarioViewSet defines a `rewind` detail route that removes some or all of a scenario's periods.
