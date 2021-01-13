---
title: Multi-player Game Tutorial
permalink: /docs/tutorials/multi-player/
layout: docs
description: Multi-player Game Tutorial
---

## Multi-player Game Tutorial

In a multi-player game, all players belonging to the same world can access the decisions and results of that world. 
The decisions and results of other worlds are hidden.
A run's leaders can access the decisions and results of all the run's worlds.

This tutorial will walk you through the creation of the multi-player `Div` game.
This game implements a model that takes a dividend and a divisor and calculates the result. 
Each world needs a player with the `Divisor` role and a player with the `Dividend` role to submit decisions in order to get a result.

The game defines two phases: `Play` and `Debrief`. During a run's `Play` phase, players can submit decisions. 
Once a leader moves a run to `Debrief` phase, players are prevented from submitting further decisions. 
During `Debrief` phase, a leader can review the final decisions and results of all worlds in the run.

The game also provides server-side input validation and reports validation errors in the UI. 

This project will demonstrate how the various pieces of Simpl interact to provide a platform upon which multi-player games can be created.

The foundation of Simpl is the Games API Service.  This service provides [REST API](http://www.django-rest-framework.org/) endpoints that allow a game
to store information about its current state in the Simpl database. 

The Model Service provides the mathematical component of a Simpl game, it interacts with the Games API and
provides messaging to the Frontend UI regarding game state.  More information can be found [here](../../overview.md).

Start by running the Simpl Games API Service following the [getting started instructions]({% link _docs/getting-started.md %}).

Next, we'll implement our Model Service providing the mathematical model for `Div`.
Last, we'll build our Frontend's user-facing pieces.

0. [Build the Multi-player Game Model Service]({% link _docs/tutorials/multi-player/modelservice.md %})
0. [Build the Multi-player Game Frontend]({% link _docs/tutorials/multi-player/frontend.md %})
