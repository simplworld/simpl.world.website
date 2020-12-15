---
title: Simpl-Games-API Endpoints
permalink: /docs/services/simple-games-api/
layout: docs
description:
---

## Simpl-Games-Api Endpoints

All endpoints are under the `/apis/` path.

### REST endpoints

The following REST endpoints are available:

* `/apis/decisions/`
* `/apis/games/`
* `/apis/hooks/`
* `/apis/periods/`
* `/apis/phases/`
* `/apis/results/`
* `/apis/roles/`
* `/apis/runs/`
* `/apis/runusers/`
* `/apis/scenarios/`
* `/apis/users/`
* `/apis/worlds/`

For more info on the specific endpoint, swagger documentation is available at the root of `simpl-games-api` (ie, if you have it running on port 8100, it will be at `http://localhost:8100/`)

### Bulk Endpoints

In addition to the RESTful endpoints, `simpl-games-api` offers endpoints to operate on multiple resources at once.

Currently, the only operation supported on these endpoints are creation (via `POST`) and deletion (via `DELETE`)

The following bulk endpoints are available:

* `/apis/bulk/decisions/`
* `/apis/bulk/periods/`
* `/apis/bulk/phases/`
* `/apis/bulk/results/`
* `/apis/bulk/roles/`
* `/apis/bulk/runs/`
* `/apis/bulk/runusers/`
* `/apis/bulk/scenarios/`
* `/apis/bulk/worlds/`

### Detail Routes

As a performance optimization, the `ScenarioViewSet` defines a `rewind` detail route that removes some or all of a scenario's periods.
