---
title: Simpl-Modelservice Overview
permalink: /docs/services/modelservice/
layout: docs
description:
---

## Simpl-Modelservice Overview

The Simpl-Modelservice provides services common to all games. These include:
 
* maintaining an in-memory copy of the game's Simpl model data as [Scopes]({% link _docs/services/modelservice/scopes.md %}).
* providing a copy of user-relevant Simpl model data to the game's UI on request.
* publishing user-relevant Simpl model data create, update and delete notifications to the game's UI.

Game modelservices use Simpl-Modelservice functionality to:

* access the Simpl database using the [Games Client]({% link _docs/services/modelservice/games-client.md %}).
* register custom RPCs on WAMP topics that are callable by the game's UI
* subscribe to WAMP topic events published by the game's UI.
