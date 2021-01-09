---
title: Simpl Overview
permalink: /docs/overview/
layout: docs
description: An overview of how Simpl works
---

## Simpl Overview

### Simpl Components

The foundation of Simpl is the [simpl-games-api](https://github.com/simplworld/simpl-games-api). A single instance of `simpl-games-api` provides a database for one or more games. 

The model service provides the mathematical component of a Simpl game. The model service contains a scope tree that defines the relationships of the game components.

The game UI provides the frontend. [simpl-ui-cookiecutter](https://github.com/simplworld/simpl-ui-cookiecutter) provides the basic layout to create a game UI. The [simpl-react](https://github.com/simplworld/simpl-react) npm module provides components for building frontend UIs using [React](https://reactjs.org) and 
[Redux](https://github.com/reduxjs/react-redux).

The browser will use the frontend service as a presentation layer, to authenticate users, and to connect to the model service in order to send and receive events through a WebSocket using [WAMP Protocol](http://wamp-proto.org/).

The model service will listen for events from the browser and, as needed, make REST API calls through the simpl-client to the `simpl-games-api` service (e.g. to store state).

Every time data is created, modified, or deleted in the `simpl-games-api` service, it will send an event to the registered game model service via HTTP Webhook.

The model service will automatically forward the event to the browser and the state will update.

`simpl-games-api` currently supports basic authentication. 
You will need to create one or more users for your modelservice(s) to use connecting to the `simpl-games-api`. 

### Simpl Defined

Roles
* Leader: oversees the game
* Player: plays the game

Single-player Versus Multi-player
* Single-player game: each player can see only his or her own decisions and results. A run’s leaders can see the decisions and results of all the run’s players.
* Multi-player game: all players belonging to the same world can see the decisions and results of that world. A run’s leaders can see the decisions and results of all the run’s worlds.

Game Terminology

<!--
* Run:
* Scenario:
-->
* Period: happens within a scenario
* Decision: a choice a player makes
* Result: the effect created by one or more decisions
* `Play` phase: players can submit decisions
* `Debrief` phase: players are prevented from submitting further decisions. The leader can review the final decisions and results of all worlds in the run.

### More Information

For more information, continue on with our tutorials and check out the service-specific documentation.


