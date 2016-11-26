---
layout: post
title: Introducing ExLogLite
cn: false
---

# Introducing ExLogLite

[ExLogLite](https://hex.pm/packages/ex_loglite) is a [Logger backend](http://elixir-lang.org/docs/stable/logger/Logger.html#module-backends) for Elixir.

When debugging with vanilla log output, the shell/REPL can be really messy if you have a lot of errors happening, and you need to scroll up until you find the first error. I find it a bit uncomfortable, and I want to a tool that can let me better view all the errors.

Since I'm working on EVE, when developing EVE we have this tool called [LogLite](https://github.com/ccpgames/EveLogLite) which is basically a GUI viewer for logs - actually this is the 3rd generation of our GUI log viewer.

So then I have this idea to make LogLite works with Elixir, and to do that you basically just need to implement a Logger Backend.

While building ExLogLite, I've learnt a lot about actually using pattern matching in function definition, and binary building in Elixir/Erlang (and IO list!). This post aims to share these knowledges and show how it's better/easier in Elixir/Erlang.

## About EVE LogLite

> LogLite is a client log viewer developed by CCP Games for their award winning MMO EVE Online.

Actually I'm not sure what the "client" means here, because we also use it on a daily basis when developing EVE. It does GUI with Qt, thus it's available across all platforms.

In the repo we also have some simple implementation of connecting different kinds of logger, such as the [python example](https://github.com/ccpgames/EveLogLite/blob/master/clients/python/__init__.py). And ExLogLite referenced these implementation a lot.

## Building the Message

As seen in the examples, we basically send some form of messages to LogLite via simple socket connection. So the first part of building ExLogLite is to actually build these messages in the same format LogLite can understand.

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

## Building the Logger

## Defects of EVE LogLite

- Unicode handling when sending LARGE_MESSAGE 
