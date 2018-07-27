---
title: Welcome to Simpl
permalink: /docs/
layout: docs
description: The Simpl framework is composed of three major parts.
---

## Welcome to Simpl

The Simpl framework is composed of three major parts:

1. The `Simpl-Games-API` service: a single instance running for all simulations, providing user information and storage for Simpl games.

2. The `Simpl-modelservice` Python framework used to build simulation model services.

3. The `Simpl-react` npm module, a module providing components for building UIs using [react](https://reactjs.org) and [redux](https://github.com/reduxjs/react-redux) that interact with a simulation model service.

These secondary framework components:

 * `simpl-authenticator` a WAMP authenticator component for Simpl.
 
 * `simpl-client` a Python client that provides asynchronous access to the data managed by the `Simpl-Games-API` service.
 
 * `simpl-users` a custom Django User model tailored to Simpl's needs.

These projects comprise an example single player simulation using Simpl:

 * `simpl-calc-model` an example of a single player simulation model service.

 * `simpl-calc-ui` an example of a browser-based UI for a single player simulation model service.
 
These projects comprise an example multi-player player simulation using Simpl:

 * `simpl-div-model` an example of a multi-player simulation model service.

 * `simpl-div-ui` an example of a browser-based UI for a multi-player simulation model service.

Simpl front ends are initially created using [cookiecutter](https://cookiecutter.readthedocs.io) using the `simpl-ui-cookiecutter` cookiecutter template.
 


