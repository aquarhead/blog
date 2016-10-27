---
layout: post
title: Introducing ExLogLite
cn: false
---

# Introducing ExLogLite

## Prelude

https://hexed.it/

Highlighted parts are where padding happens

```python
>>> msg = _Message()
>>> msg.type = _MessageType.CONNECTION_MESSAGE
>>> msg.body.connection.version = VERSION
>>> msg.body.connection.pid = 12312
>>> msg.body.connection.machine_name = 'dfa'
>>> msg.body.connection.executable_path = 'qqq'
>>> buffer(msg)
<read-only buffer for 0x10a371cb0, size -1, offset 0 at 0x10a6e1670>
>>> q = buffer(msg)[:]
>>> len(q)
344
>>> 64+256+4+8+4
336
>>> f = open("msg", "wb")
>>> f.write(q)
>>> f.close()
```
