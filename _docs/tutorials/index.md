---
title: Simpl Calc Tutorial
permalink: /docs/tutorials/
layout: docs
description:
---

# Simpl Calc Tutorial

This tutorial will walk you through the creation of the Simpl Calc game.
This game implements a model that takes a number and adds it to a current total.
This project, though basic, will demonstrate how the various pieces of Simpl interact to provide a platform
upon which single player games can be created.

The foundation of Simpl is the Games API Service.  This service provides API endpoints that allow the game
to store information regarding a game and its current state in the database.
The Model Service provides the mathematical component of a Simpl game, it interacts with the Games API and
provides messaging back to the Frontend UI regarding game state.  More information can be found [here](../overview.md).

We'll start by setting up our API service. Next we'll implement our Model Service, giving us our mathematical model for Simpl Calc.
Lastly we'll build the user-facing pieces.

0. [Build the Simpl-Games-API service](games-api.md)
0. [Build the Single Player Model Service](modelservice.md)
0. [Build the Single Player Frontend UI](frontend.md)
