---
layout: post
title: "Simpl-y Magic: Automatic Browser Page Updates"
date: 2017-09-01 12:10:00 -0500
categories: News
author: Jane Eisenstein
author_photo: /assets/img/authors/jane-eisenstein.png
excerpt: |
    Simpl's architecture ensures a game user’s browser page is always up to date.
---

In multiplayer web-based games, all users should be able to see up-to-date game data without having to manually refresh their browsers. For example, players need to be notified when the game has moved from a phase in which they can submit decisions to a phase in which they cannot submit decisions. Monitoring the game state for such changes is often handled by the simulation's front-end code.

One of the real pleasures of developing simulations using the[ Learning Lab-authored](http://simulations.wharton.upenn.edu/2017/01/13/simpl/) Simpl framework is never needing to request fresh data in front-end code. That's because Simpl’s architecture ensures a game user's browser page is *always* up to date. Curious how we managed to pull that off? Then keep reading!

First, it's important to understand that each Simpl game comprises three components:

* **Simpl-Games-API** (a service shared with other games that maintains the Simpl database)

* **Model Service** (defines and runs the game's simulation model)

* **Front-end Server** (provides the game's user interface assets to the browser)

![image alt text](/assets/img/blog/simply-magic-automatic-browser-page-update
/image_0.png){: width="600" }

*Architecture of a Simpl game*

Our **Simpl-Games-API** service manages the Simpl database. It provides a[ REST API](http://www.django-rest-framework.org) used by the game's model service.

The game's model service defines the game's simulation model and handles running the simulation and database updates. It is implemented in Python using classes provided by our **Simpl-Modelservice** package.

The game's front-end user interface code is implemented in Javascript using Simpl game front-end functions provided by our **Simpl-React** JavaScript library (built using[ React](https://facebook.github.io/react/) and[ Redux](https://code-cartoons.com/a-cartoon-intro-to-redux-3afb775501a6#.pka6i965c)).

These Simpl game components work together in concert to ensure that game users consistently see the current state-of-the-game data stored in the Simpl database.

Here's how it works: Each time the model service updates the database using the Simpl-Games-API's REST API, a [webhook](https://pypi.python.org/pypi/thorn/) is triggered that notifies Simpl-Modelservice functions of the update. Simpl-Modelservice code then pushes an update notification to each game user's browser via [WAMP](http://wamp-proto.org/why/). There, Simpl-React code handles updating the browser's Redux store state, automatically updating the React components.

And there you have it — the user is automagically guaranteed to see fresh data, without game authors having to write a line of code! It works like magic, but it's actually quite *Simpl*...

For more details, please see our [Simpl Documentation]({% link _docs/index.md %}).

