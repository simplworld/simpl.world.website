---
title: Simpl Overview
permalink: /docs/overview/
layout: docs
description: Simpl Overview description
---

## Overview

In practice, there will always be a single instance of The `Simpl-Games-API` service, and for every simulation you
will write a model service, and a frontend service. The browser will use the frontend service for the static assets
(HTML and JS), to authenticate users,  and to connect to the model service in order to receive/send events through the WebSocket.

The `Simpl-Games-API` service will send events to your model service via HTTP Webhooks, and they will be
automatically forwarded to the browser.

Your model service will send messages to the browser or listen to messages from it, and will make
REST API calls to the `Simpl-Games-API` service as needed (eg to store state).


```plain
    +-------------------------------------------------+
    |                `Simpl-Games-API`                |
    +-------------------------------------------------+
              | A                         | A
     Webhooks | |  REST API      Webhooks | |  REST API
              V |                         V |
   +-------------------+          +-------------------+
   | modelservice sim1 |          | modelservice sim2 |
   +-------------------+          +-------------------+
        A           A                 A           A
        | WAMP      | WAMP            | WAMP      | WAMP
        V           V                 V           V
+-----------+  +-----------+  +-----------+  +-----------+
| Browser 1 |  | Browser 2 |  | Browser 3 |  | Browser 4 |
+-----------+  +-----------+  +-----------+  +-----------+
     A               A             A               A
     | Static        | Static      | Static        | Static
     | assets        | assets      | assets        | assets
   +-------------------+         +-------------------+
   |   frontend sim1   |         |   frontend sim2   |
   +-------------------+         +-------------------+


```

## The WAMP Layer

The communication between the modelservice and the browser happens via Websocket by using the [WAMP Protocol](http://wamp-proto.org/).

Thw WAMP Protocol is mainly composed of two patterns: publish/subscribe (**PubSub** in short) and call/register (or **RPC**).

### PubSub vs RPC

The main difference between PubSub and RPC is that a _call_ to a registered procedure returns a value,
where _publishing_ to a topic does not. Both are asynchronous operations, but with RPC you can wait on the returned value
to tell when the operation has completed.

As a rule of thumb: use RPC if you need to get data from the modelservice or wait until the procedure has completed before proceeding.
Otherwise you can use PubSub.
