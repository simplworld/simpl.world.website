# Custom RunUser

You may want to provide your custom `RunUser` scope, implementing the procedures and subscribers necessary for your specific situation.

Your custom `RunUser` scope will need to be registered with your top-level scope, so scopes across your game can know about it.

In order to do that, you can use the `runuser_class` attribute:

```python
import random
from modelservice.games import Run, RunUser
from modelservice.games import game, register


class ZSRunUser(RunUser):
    @register
    def throw_dice(self):
        return random.randint(1, 6)


@game('zero-sum')
class ZSRun(Run):
    runuser_class = ZSRunUser
```