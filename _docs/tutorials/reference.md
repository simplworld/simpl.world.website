---
title: Simpl Reference
permalink: /docs/tutorials/reference/
layout: docs
description:
---

## Reference

### Bringing Development Environment Online

Provided you've completed the Simpl Calc tutorial, the following steps outline bringing everything back
online from a freshly restarted Vagrant

Open up a terminal window, navigate to your Vagrant folder, and bring up the Vagrant box, `vagrant up`

Start Simpl Games API

```bash
$ vagrant ssh
$ workon simpl-games-api
$ cd /vagrant/projects/simpl-games-api
$ ./manage.py runserver 0.0.0.0:8100
```

Open up a new terminal window, navigate to your Vagrant folder.

Start Model Service

```bash
$ vagrant ssh
$ workon simpl-calc-model
$ cd /vagrant/projects/simpl-calc-model
$ ./manage.py run_modelservice
```

Open up a new terminal window, navigate to your Vagrant folder.

Start Simpl Calc UI

```bash
$ vagrant ssh
$ workon simpl-calc-ui
$ cd /vagrant/projects/simpl-calc-ui
$ ./manage.py runserver 0.0.0.0:8000
```
