---
title: Building the Simpl-Games-API service
permalink: /docs/tutorials/games-api/
layout: docs
description:
---

# Building the Simpl-Games-API service

## Prerequisites

The Simpl Calc tutorial assumes that the following software is installed:

* [Simpl Python-Dev Development Environment (CentOS 7) Vagrant Box](https://github.com/simplworld/python-vagrant-centos7)
	* PostgreSQL >= 9.6
	* Python >= 3.6

If you do not have these prerequisites installed, please follow the below steps.

Clone the Simpl `python-vagrant-centos7` image:

```bash
$ git clone ssh://git@github.com:simplworld/python-vagrant-centos7.git
```

Clone the simpl-games-api repository in the Vagrant image's project directory:

```bash
$ cd python-vagrant-centos7/projects
git clone git@github.com:simplworld/python-vagrant-centos7.git
```

Start Vagrant and connect:

```bash
$ vagrant up
$ vagrant ssh
```

You are now ready to install the Simpl Games API server following the instructions in the simpl-games-api README



Start the web server. Please note that the server should run on port 8100:

```bash
$ ./manage.py runserver 0.0.0.0:8100
```

This is all you need for your local Simpl-Games-API instance.

You can now head over to the [Model Service Tutorial](./modelservice.md).
