---
layout: post
title: "Understanding the Simpl Data Model"
date: 2018-06-20 14:38:00 -0400
categories: Design
author: Jane Eisenstein
author_photo: /assets/img/authors/jane-eisenstein.png
excerpt: |
    The Simpl data model was designed to support developing gamified simulations with as much flexibility as possible.
---

## Background

The Simpl framework was developed by the [Alfred West Jr. Learning Lab](http://simulations.wharton.upenn.edu/) at the Wharton School
of the University of Pennsylvania. The Learning Lab’s mission is to improve the way Wharton students learn by engaging them with
cutting edge digital learning tools (usually gamified simulations) developed in partnership with Wharton faculty.

At Wharton, games are run as part of a class or other event. A group of students plays a game by entering decisions for one or more
simulated time periods over minutes, hours or days of real time.
The students then participate in a debrief session with their teacher during which they cover what can be learned from the game results.

In some games, players are grouped into teams to either collaborate or compete with other team members. These are **multi-player** games.
In other games, player decisions and results are independent of each other. These are **single-player** games.

## Model Classes

The Simpl data model was designed to support developing both multi-player and single-player games with as much flexibility as possible.
To provide extra flexibility, each model class has a data property for storing JSON data unique to a specific game.

A **Game** encapsulates all the functionality of a simulation. Each Game defines simulation model logic and provides a web-based user interface.

A **Run** is an instance of a Game created to be used in a class or other event. The state and data in one Run has no bearing on other Runs.

A Simpl **User** is a [Django](https://www.djangoproject.com) user identified by email address with some additional fields.
Users can participate in several games by being assigned to Runs of the Games.

A **RunUser** represents the one to one relationship between a game Run and a User.
Each RunUser represents either a leader or a player in the run. Each Run may have one or more leader RunUsers and one or more player RunUsers.

A Game can have one or more Run **Phases** that determine the features available as a Run moves from one phase to the next.
Simple Games might need only one Phase (e.g. *Play*), while more complex Games may require several (e.g. *Setup*, *Players Prepare*, *Play*, and *Debrief*).

A Game can have one or more player **Roles**. A player’s role might determine the background information they see and the types of decisions they can make.

A **World** groups RunUsers into a team whose combined decisions are used to step the Game’s simulation model.
Each World's decisions and results are independent of other World's decisions and results.

A **Decision** is a set of inputs for stepping the simulation model. One or more Decisions may be needed to step the simulation model.
Decisions may be assigned to specific player Roles.

A **Result** is a set of outputs from stepping the simulation model. Results may be assigned to specific player Roles.

A **Period** represents inputs and outputs for a step of the simulation model.

A **Scenario** encapsulates a set of Decisions and Results possibly over several Periods for a RunUser or for a World.

These models can be used to create a wide variety of simulation-based games.


## Designing a Multi-player Game

How might we use Simpl models to re-implement the Learning Lab's [Macrosim](http://simulations.wharton.upenn.edu/solutions/macrosim/)  game?
Macrosim players practice on their own over several days making  fiscal and monetary policy decisions that effect a country's economy.
They  then play a tournament during class as part of a team.
Each Macrosim tournament team comprises some students playing the role of monetary policy maker and other students playing the role of fiscal policy maker.

To re-implement Macrosim as a Simpl game, we would create a single *macrosim* Game instance with  *MonetaryPolicyMaker* and *FiscalPolicyMaker* Roles.
The *macrosim* Game would also need three Phases: *Setup*, *Practice*, *Tournament*.

During the *Setup* phase of a *macrosim* Run, leaders assigned to the Run would be able to log into the game to configure the game's settings and
add students to the Run as player RunUsers with an assigned World and Role.

After moving a  *macrosim* Run to *Practice*, students would be able to log in and make *MonetaryPolicyMaker* and *FiscalPolicyMaker* Decisions
and see their Results over several Periods using private RunUser Scenarios.

The tournament  leader would move a *macrosim* Run from *Practice* to the *Tournament* phase to bring an end to student practice and open a
tournament of country (aka World) economies competing for the best final Results.
During the tournament, a *MonetaryPolicyMaker* player and a *FiscalPolicyMaker* player in each World would enter Decisions for their World's Scenario.
The tournament leader would control how many Periods the tournament runs, when players could enter Decisions, when the Macrosim simulation model runs the Decisions, and discuss the simulation Results of all Worlds.
Logged in players would only be able to see the Decisions and Results of their assigned World.


## Simpl Django Model Diagram


![image alt text](/assets/img/blog/understanding-the-simpl-data-model/models.png)














