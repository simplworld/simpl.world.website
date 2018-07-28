---
title: Single Player Game Tutorial
permalink: /docs/tutorials/single-player/
layout: docs
description: Single Player Game Tutorial
---

## Single Player Game Tutorial

This tutorial will walk you through the creation of the single player `Calc` game.
This game implements a model that takes a number and adds it to the current total.
This project, though basic, will demonstrate how the various pieces of Simpl interact to provide a platform
upon which single player games can be created.

The foundation of Simpl is the Games API Service.  This service provides [REST API](http://www.django-rest-framework.org/) endpoints that allow a game
to store information about its current state in the Simpl database. 

The Model Service provides the mathematical component of a Simpl game, it interacts with the Games API and
provides messaging to the Frontend UI regarding game state.  More information can be found [here](../../overview.md).

Start by running the Simpl Games API Service following the [getting started instructions]({% link _docs/getting-started.md %}).

Next, we'll implement our Model Service providing the mathematical model for `Calc`.
Last, we'll build our Frontend's user-facing pieces.

0. [Build the Single Player Game Model Service]({% link _docs/tutorials/single-player/modelservice.md %})
0. [Build the Single Player Game Frontend({% link _docs/tutorials/single-player/frontend.md %})
