---
title: Welcome to Simpl
permalink: /docs/
layout: docs
description: The Simpl framework is composed of three major parts.
---

## Welcome to Simpl

The Simpl framework is composed of three major parts:

1. The [simpl-games-api]<!--(https://github.com/simplworld/simpl-games-api)--> service: a single instance running for all simulations, providing user information and storage for Simpl games.

2. The [simpl-modelservice]<!--(https://github.com/simplworld/simpl-modelservice)--> Python framework used to build simulation model services.

3. The [simpl-react]<!--(https://github.com/simplworld/simpl-react)--> npm module providing components for building frontend UIs using [react](https://reactjs.org) and 
[redux](https://github.com/reduxjs/react-redux) that interact with a simulation model service.

These secondary framework components:

 * [simpl-authenticator]<!--(https://github.com/simplworld/simpl-authenticator)--> a WAMP authenticator component for Simpl.
 
 * [simpl-client]<!--(https://github.com/simplworld/simpl-client)--> a Python client that provides asynchronous access to the data managed by the `simpl-games-api` service.
 
 * [simpl-users]<!--(https://github.com/simplworld/simpl-users)--> a custom Django User model tailored to Simpl's needs.

These projects comprise an example single player simulation using Simpl:

 * [simpl-calc-model]<!--(https://github.com/simplworld/simpl-calc-model)--> an example of a single player simulation model service.

 * [simpl-calc-ui]<!--(https://github.com/simplworld/simpl-calc-ui)--> an example of a browser-based UI for a single player simulation model service.
 
These projects comprise an example multi-player player simulation using Simpl:

 * [simpl-div-model]<!--(https://github.com/simplworld/simpl-div-model)--> an example of a multi-player simulation model service.

 * [simpl-div-ui]<!--(https://github.com/simplworld/simpl-div-ui)--> an example of a browser-based UI for a multi-player simulation model service.

Simpl front ends are initialized with [cookiecutter](https://cookiecutter.readthedocs.io) using 
the [simpl-ui-cookiecutter]<!--(https://github.com/simplworld/simpl-ui-cookiecutter)--> cookiecutter template.
 


