---
title: Docker Installation
permalink: /docs/installation/
layout: docs
description: How to install Simpl using Docker
---

## Docker Installation

You will need to have [Docker](https://www.docker.com/) installed.

At the end of this tutorial, you will have successfully installed and played a single-player `Calc` game.

In order to play a game, you will need to install and run `simpl-games-api`, a game model service, and a game UI.

For every game, you will write the game model service, and the game UI. 

For this tutorial, we will use [simpl-calc-model](https://github.com/simplworld/simpl-calc-model) and [simpl-calc-ui](https://github.com/simplworld/simpl-calc-ui)

Clone the simpl-games-api, simpl-calc-model, and simpl-calc-ui repositories:

```bash
git clone git@github.com:simplworld/simpl-games-api.git
git clone git@github.com:simplworld/simpl-calc-model.git
git clone git@github.com:simplworld/simpl-calc-ui.git
```

The `simpl-games-api` should always be installed first and will run the entire time.

`cd` into the `simpl-games-api` repo and start Docker.

```bash
cd simpl-games-api
docker-compose up
```

Open another terminal window. In that window, use a `make` script to log into the container, which is named `api`.

```bash
make shell
```

A `superuser` and `staff user` will need to be created. 

`superuser` can be used to access the admin. `staff user` can be used to access the Simpl API.

Create a `superuser` when prompted: "We found no superusers, please set one up below"

`superuser` can have any email address and password.

Create a `staff user` when prompted: "We found no staff users, you will need one of these for your model service to connect to the Simpl API"

`staff user` must use the email address `simpl@simpl.world` and password `simpl`, which correspond to the email address and password in the model service `settings.py` file.

`simpl-games-api` is now up and running. You can now exit and close that terminal window.

```bash
exit
```

Open another terminal window and `cd` into `simpl-calc-model`. We will now get `simpl-calc-model` up and running. `docker-compose up` will need to be run twice. The first time you run it, you will see an error, because the model service will expect a game, but the game doesn't exist yet.

```bash
cd simpl-calc-model
docker-compose up
```

In your browser, open up `http://localhost:8100/admin/`. The Django admin login page should appear.

You can now login using either the `superuser` or `staff user` credentials.

Create another terminal window. You will now run `simpl-calc-model` and create the game, including the database.

```bash
docker-compose run --rm model.backend bash
./manage.py create_default_env
```

You can now exit and shut down the Docker container.

```bash
exit
docker-compose down
```

Restart the Docker container. The game now exists and can be run in-memory.

```bash
docker-compose up
```

`cd` into the `simpl-calc-ui` repo and run the UI.

```bash
cd simpl-calc-ui
docker-compose up
```

All three containers are now running on the same private network using Docker.

In your browser, open up `http://0.0.0.0:8000/`. The `simplcalc` game should appear.

To shut down the game, run `docker-compose down` in the terminal windows running the `simpl-calc-ui`, `simpl-calc-model`, and `simpl-games-api` containers. 

```bash
docker-compose down
```