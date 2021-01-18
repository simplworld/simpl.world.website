---
title: Single-player Game Demo
permalink: /docs/demo/
layout: docs
description: Run a Simpl single-player game demo using Docker
---

## Single-player Game Demo

Although we recommend using Docker, the non-Docker [Single-player Game Demo instructions]({% link _docs/getting-started.md %}) are available for a limited time.

To complete this tutorial, you will need to have [Docker](https://www.docker.com/) installed.  

At the end of this tutorial, you will have successfully installed and played a single-player `Calc` game.

In order to play a game, you will need to install and run `simpl-games-api`, a game model service, and a game UI.

For every game, you will write the game model service, and the game UI. 

For this tutorial, we will use [simpl-calc-model](https://github.com/simplworld/simpl-calc-model) and [simpl-calc-ui](https://github.com/simplworld/simpl-calc-ui)

First, clone the [simple-games-api](https://github.com/simplworld/simpl-games-api) repository:

```bash
git clone https://github.com/simplworld/simpl-games-api
```

The `simpl-games-api` should always be installed first and will run the entire time.

`cd` into the `simpl-games-api` repo and start Docker.

```bash
cd simpl-games-api
docker-compose up
```

When the `simpl-games-api` initialization is complete, open another terminal window. In that window, use a `make` script to log into the container, which is named `api`.

```bash
make shell
```

You will see in the window that you are now logged into the container.

```bash
Creating simpl-games-api_api_run ... done
root@b2397c235f68:/code# 
```

Create a `superuser` and `staff user` by running `./manage.py simpl_setup` 

```bash
root@b2397c235f68:/code# ./manage.py simpl_setup
```

`superuser` is used to access the data in the Simpl API's Django admin. `staff user` is used by Simpl model services to connect to the Simpl API.

Create a `superuser` when prompted. `superuser` can have any email address and password.

```bash
=== Setup Simpl API ===

We found no superusers, please set one up below
--- Creating a superuser ---
Please enter email address for superuser: 
```

Create a `staff user` when prompted. `staff user` must use the email address `simpl@simpl.world` and password `simpl`, which correspond to the email address and password in the model service `settings.py` file.

```bash
We found no staff users, you will need one of these for your model service to connect to the Simpl API
This is typically setup as a setting `SIMPL_GAMES_AUTH` from the environment variables `SIMPL_USER` and `SIMPL_PASS`

--- Creating a staff user ---
Please enter email address for staff user: 
```

You've created the Django users needed to run the game. You can now exit and close that window.

```bash
exit
```

In your browser, open up `http://localhost:8100/admin/`. The Django admin login page should appear.

You can now login using either the `superuser` or `staff user` credentials.

With the `simpl-games-api` still running in the first window, open another window and `cd` back into your user directory.

```bash
cd ..
```

Clone the [simple-calc-model](https://github.com/simplworld/simpl-calc-model) repository and `cd` into it.

```bash
git clone https://github.com/simplworld/simpl-calc-model
cd simpl-calc-model
```

We will now get `simpl-calc-model` up and running. 

```bash
docker-compose up
```

`docker-compose up` will need to be run twice. The first time you run it, you will see an error, because the model service will expect a game, but the game doesn't exist yet.

```bash
#snippet
genericclient_base.exceptions.ResourceNotFound: No `games` found for {'slug': 'simpl-calc'}
```

Create a third window. You will now run `simpl-calc-model` to create the game in the database.

```bash
docker-compose run --rm model.backend bash
```

You will see in the window that you are now logged into the container.

```bash
Creating simpl-calc-model_model.backend_run ... done
root@9010e0c4bd2b:/code#                                                                 
```

Set up the game by running `./manage.py create_default_env` 

```bash
root@b2397c235f68:/code# ./manage.py create_default_env
```

You can now exit, shut down the Docker container, and close that window.

```bash
exit
docker-compose down
```

Go back to the window that `simpl-calc-model` has been running in. Restart the Docker container. Because the game now exists, the error will be gone.

```bash
docker-compose up
```

With the `simpl-games-api` still running in the first window, and the `simpl-calc-model` running in another window, open a third window and `cd` back into your user directory.

```bash
cd ..
```

Clone the [simple-calc-ui](https://github.com/simplworld/simpl-calc-ui) repository and `cd` into it.

```bash
git clone https://github.com/simplworld/simpl-calc-ui
cd simpl-calc-ui
```

Run `docker-compose up` to run the UI.

```bash
docker-compose up
```

All three containers are now running on the same private network using Docker.

In your browser, open up `http://localhost:8000/`. The `simplcalc` game should appear.

To shut down the game, run `docker-compose down` in the windows running the `simpl-calc-ui`, `simpl-calc-model`, and `simpl-games-api` containers. 

```bash
docker-compose down
```