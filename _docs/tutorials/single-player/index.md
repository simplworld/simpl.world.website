---
title: Single-player Game Tutorial
permalink: /docs/tutorials/single-player/
layout: docs
description: Single-player Game Tutorial
---

## Single-player Game Tutorial

In a single-player game, each player sees only his or her own decisions and results. 
The decisions and results of other players are hidden. 
Only a run's leaders can access the decisions and results of all the run's players.

This tutorial will walk you through the creation of the single-player `Calc` game.
This game implements a model that takes a number and adds it to the current total.
The game has a single `Play` phase and does not define any player roles.
This project, though basic, will demonstrate how the various pieces of Simpl interact to provide a platform
upon which single-player games can be created.

For an overview of how Simpl works, see [Simpl Overview]({% link _docs/overview.md %}).

Start this tutorial by running the Simpl Games API service using the instructions in [Getting Started]({% link _docs/getting-started.md %}).

First, we'll implement our [Single-player Game Model Service]({% link _docs/tutorials/single-player/modelservice.md %}), providing the mathematical model for `Calc`.

Then, we'll build our [Single-player Game Frontend]({% link _docs/tutorials/single-player/frontend.md %}).