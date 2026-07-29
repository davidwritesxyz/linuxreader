---
title: 'Toggle PoE on a Juniper Switch'
description: 'How to toggle PoE on a Juniper switch.'
---

```bash
Configure

set poe interface ge-0/0/0 disable

commit

rollback 1

commit
```
