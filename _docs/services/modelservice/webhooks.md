# Registering functions with hooks

Inside your Django app, create a file called `webhooks.py` with the following content:

```python
from modelservice.webhooks import hook

@hook('SET_PHASE')
def myfunc(payload):
    do_something()
```

`webhooks.py` modules are automatically discovered and imported.

You can register multiple functions with the same event, but the order in which they are executed is undetermined.
