---
title: Simpl-Modelservice Overview
permalink: /docs/services/modelservice/overview/
layout: docs
description: How Simpl-Modelservice Works
---

## Simpl-Modelservice Overview

The Simpl-Modelservice provides services common to all games. These include:
 
* maintaining an in-memory copy of the game's Simpl model data as [Scopes]({% link _docs/services/modelservice/scopes.md %}).
* providing a copy of user-relevant Simpl model data to the game's UI on request.
* publishing user-relevant Simpl model data create, update and delete notifications received 
via Simple-Games service Webhooks to the game's UI.

Game modelservices use Simpl-Modelservice functionality to:

* access the Simpl database using the [Games Client]({% link _docs/services/modelservice/games-client.md %}).
* register custom RPCs on WAMP topics that are callable by the game's UI
* subscribe to WAMP topic events published by the game's UI.

### The WAMP Layer

The browser and model service communicate via Websocket using the [WAMP Protocol](http://wamp-proto.org/).

The WAMP Protocol is mainly composed of two patterns: publish/subscribe (**PubSub** for short) and call/register (or **RPC**).

#### PubSub vs RPC

The main difference between PubSub and RPC is that a _call_ to a registered procedure returns a value,
where _publishing_ to a topic does not. Both are asynchronous operations, but with RPC you can wait on the returned value to tell when the operation has completed.

As a rule of thumb: use RPC if you need to get data from the modelservice or wait until the procedure has completed before proceeding. Otherwise you can use PubSub.

