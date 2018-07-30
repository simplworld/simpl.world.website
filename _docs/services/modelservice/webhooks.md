---
title: Registering Functions with Hooks
permalink: /docs/services/modelservice/webhooks/
layout: docs
description:
---

## Registering Functions with Hooks

The Run Scope class provides these hooks for adding custom handling for webhook notifications:

```python
    async def on_runuser_deleted(self, payload):
        """
        Override this hook to provide custom behavior after deleting
        a runuser from a run
        """
        
     async def on_runuser_created(self, runuser_id):
         """
         Override this hook to provide custom behavior after adding
         a runuser to a run
         """ 
    async def on_advance_phase(self, next_phase):
        """
        Override this hook to provide custom behavior after advancing the run
        to the next phase.
        """

    async def on_rollback_phase(self, previous_phase):
        """
        Override this hook to provide custom behavior after reverting the run
        to the previous phase.
        """
```


If you need custom behaviors for other webhook notifications, 
create a file called `webhooks.py` in your your `game` app that defines `hook` functions.  For example:

```python
from modelservice.webhooks import hook

@hook('SET_PHASE')
def myfunc(payload):
    do_something()
```

`webhooks.py` modules are automatically discovered and imported.

You can register multiple functions with the same event, but the order in which they are executed is undetermined.


