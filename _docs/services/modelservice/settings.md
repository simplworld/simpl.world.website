---
title: Settings
permalink: /docs/services/modelservice/settings/
layout: docs
description:
---

## Settings

### Required settings

```python
CALLBACK_URL = os.environ.get('CALLBACK_URL', 'http://{hostname}:{port}/callback')
SIMPL_GAMES_URL = os.environ.get('SIMPL_GAMES_URL', 'http://localhost:8100/apis')
SIMPL_GAMES_AUTH = ('system', 'System!1')
ROOT_TOPIC = 'com.example'
```
