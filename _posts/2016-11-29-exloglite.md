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

By looking at the python client example or `logmodel.cpp` inside LogLite project, we need to build some C struct following its definition. I used [HexEd.it](https://hexed.it/) to figure out how some zero padding magic happend:

![byte formats](/static/exloglite/message_bytes.png)

(Left is a connection message, right is a simple message. Highlighted parts are where padding happens)

As seen in the examples, we basically send some form of messages to LogLite via simple socket connection. So the first part of building ExLogLite is to actually build these messages in the same format LogLite can understand.

In ExLogLite, I have `ExLogLite.LogModel` to build these actual bytes (or bitstring in BEAM world). The only function that should be public is `build_message`, which is called by the actual Logger Backend.

```elixir
def build_message(:connection, {pid, machine_name, exec_path}) do
  [
    << 0::32-little, 0::32 >>,
    << @version::32-little, 0::32 >>,
    << pid::64-little >>,
    build_binary_chars(machine_name, 32),
    build_binary_chars(exec_path, 260),
    << 0::size(28)-unit(8) >>
  ]
end
def build_message(typ, msg) do
  [ build_type_part(typ) | build_text_message_part(msg) ]
end
```

First thing to notice, what we return in both definitions are just a list. To be specific, it’s an IO List, you should really read [this post](https://www.bignerdranch.com/blog/elixir-and-io-lists-part-1-building-output-efficiently/) which explains the benefits in detail. But for now you just need to know Erlang - thus Elixir - treats a list-ish binary stuff just like a single binary. And it’s perfectly fine if all you need to do is to send this stuff over the wire.

To actually build a bitstring, we use the `<<args>>/1` Special Form in Elixir, you can explore its capabilities from the [detailed documentation](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#%3C%3C%3E%3E/1). Basically we can use some modifiers after `::` to specify the format and how to layout the thing before it to bitstring.

Next, notice we’re using pattern matching while defining this function. Instead of scattering code with same responsibility in several places or using `if/else`, we write different code for different (types of) arguments. I find this style of writing code could deeply change how I approach problems, and how to organize my solutions. 

Instead of having deep hierarchies of `if/else` for complex requirements, we divide the steps into smaller, simpler functions, and use pattern matching to seperate all the different flows.

```elixir
defp build_type_part(:simple), do: << 1::32-little, 0::32 >>
defp build_type_part(:large), do: << 2::32-little, 0::32 >>
defp build_type_part(:continuation), do: << 3::32-little, 0::32 >>
defp build_type_part(:continuation_end), do: << 4::32-little, 0::32 >>

defp build_text_message_part({timestamp, severity, module, channel, message}) do
  [
    << timestamp::64-little >>,
    << severity::32-little >>,
    build_binary_chars(module, 32),
    build_binary_chars(channel, 32),
    build_binary_chars(message, 256),
    << 0::32 >>
  ]
end
```

These two functions should be pretty familiar by now, `build_type_part` applies pattern matching again. And `build_text_message_part` uses `<<args>>/1` to build a IO List that simply concats to the type part (bitstring) thus returns another IO List.

```elixir
@doc """
Build binary in the same format as char array in C from a string.
Trim `str` if exceeds `max_len`, pad 0 if not long enough. Count in `byte_size`
"""
def build_binary_chars(str, max_len) when byte_size(str) == max_len, do: << str::bytes >>
def build_binary_chars(str, max_len) when byte_size(str) > max_len, do: << binary_part(str, 0, max_len)::bytes >>
def build_binary_chars(str, max_len) do
  pad_len = max_len - byte_size(str)
  << str::bytes, 0::size(pad_len)-unit(8) >>
end
```

The only remaining part is a function to make a bitstring with defined size out of an arbitary string. Relying on pattern matching again, we just need to deal with 3 kinds of strings:

1. If the byte size if perfectly equal to `max_len`, we return it as a bitstring directly
2. If larger we take just the part upto `max_len`
3. If smaller we pad more 0s to make it the desired size

This function should really be private, but I also wrote some unit test as I went through various iterations of its implementation:

```elixir
test "`build_binary_chars` trim longer string" do
  assert LogModel.build_binary_chars("dfa", 2) == <<"df">>
end

test "`build_binary_chars` extend shorter string" do
  assert LogModel.build_binary_chars("dfa", 4) == <<"dfa", 0::8>>
end

test "`build_binary_chars` should handle unicode correctly" do
  assert LogModel.build_binary_chars("測試...", 1) == <<230>>

  assert LogModel.build_binary_chars("測試", 7) == << "測試", 0 >>
end
```

## Building the Logger

Now we can build the bitstring for all kinds of messages we want to send to LogLite, we just need to implement the actual Logger Backend.

Looking at the python client, basically it establishes a TCP connection to LogLite which runs a server at `0xCC9` port, and sends a connection message. Then for each log we send actual messages to it.

To implement a Logger Backend in Elixir, we basically need to implement the `GenEvent` behaviour. With `GenEvent` we can setup the TCP connection and send the connection message in `init/1`, actual logging are handled in `handle_event`.

The `init/1` is pretty trivial, let’s focus on what happens for an actual log:

```elixir
def handle_event({lv, _gl, {Logger, msg, ts, md}}, %{socket: socket} = state) do
  if byte_size(msg) <= 255 do
    send_simple_msg(socket, lv, msg, ts, md)
  else
    send_large_msg(socket, lv, msg, ts, md)
  end

  {:ok, state}
end

defp send_simple_msg(socket, lv, msg, ts, md) do
  log_msg = LogModel.build_message(
    :simple,
    {
      convert_timestamp(ts),
      level_to_severity(lv),
      get_loglite_module(md),
      get_loglite_channel(md),
      msg
    })

  :gen_tcp.send(socket, log_msg)
end

defp send_large_msg(socket, lv, msg, ts, md) do
  msg_parts = split_large_msg(msg)

  begin_send_multiparts(socket, lv, ts, md, msg_parts)
end
```

For a simple message, the code is indeed simple and straightforward. However for large messages, I tried a more functional way instead using `while` like in the python client.

First we transform the big message into a list of appropriately sized bitstrings, using the common practice of tail recursion:

```elixir
defp split_large_msg(msg), do: divide_bytes(msg, 255, []) |> Enum.reverse

defp divide_bytes(bytes, div_len, acc) when byte_size(bytes) < div_len do
  [bytes | acc]
end
defp divide_bytes(bytes, div_len, acc) do
  size = byte_size(bytes)
  this_part = binary_part(bytes, 0, div_len)
  left_bytes = binary_part(bytes, div_len, size - div_len)

  divide_bytes(left_bytes, div_len, [this_part | acc])
end
```

Next we kick off sending those messages with a `:large` message:

```elixir
defp begin_send_multiparts(socket, lv, ts, md, [first_part | msg_parts]) do
  first_msg = LogModel.build_message(
    :large,
    {
      convert_timestamp(ts),
      level_to_severity(lv),
      get_loglite_module(md),
      get_loglite_channel(md),
      first_part
    }
  )

  :gen_tcp.send(socket, first_msg)

  send_multiparts_cont(socket, lv, ts, md, msg_parts)
end
```

Then we use pattern matching and tail recursion again for the rest, in correct types:

```elixir
defp send_multiparts_cont(socket, lv, ts, md, [last_part]) do
  last_msg = LogModel.build_message(
    :continuation_end,
    {
      convert_timestamp(ts),
      level_to_severity(lv),
      get_loglite_module(md),
      get_loglite_channel(md),
      last_part
    }
  )

  :gen_tcp.send(socket, last_msg)
end
defp send_multiparts_cont(socket, lv, ts, md, [this_part | msg_parts]) do
  this_msg = LogModel.build_message(
    :continuation,
    {
      convert_timestamp(ts),
      level_to_severity(lv),
      get_loglite_module(md),
      get_loglite_channel(md),
      this_part
    }
  )

  :gen_tcp.send(socket, this_msg)

  send_multiparts_cont(socket, lv, ts, md, msg_parts)
end
```

And that's it! I ignored some helper functions for converting timestamps and log level etc.. into the format LogLite expects, but so far we've covered all the important bits of ExLogLite.

## See It in Action
Fire up `iex -S mix` inside the ExLogLite project, add the backend and try some logging:

```elixir
iex(1)> :application.start(:logger)
:ok
iex(2)> require Logger
Logger
iex(3)> Logger.add_backend(ExLogLite)
{:ok, #PID<0.142.0>}
iex(4)> Logger.error("shit!")
```

This is how it looks in LogLite:

![LogLite](/static/exloglite/LogLite.png)

## Lastly & Future Improvements

Actually I've had the idea of incorporating EVE LogLite into Elixir for quite some time, since it's a tool I use on a daily basis. However I basically wrote all of ExLogLite in a big hurry for my - quite possibly - last Elixir Shanghai meetup. Back then, I was doing all the things for relocation from Shanghai to Reykjavik where actual EVE development happens. I feel really bad for not preparing a good talk for my last Elixir Shanghai meetup, which I really enjoy. But I'm simply not able to spare any more time besides all my hurried and messy relocation stuff.

I'd like to keep improving ExLogLite, and EVE LogLite as well if possible, such as:

- Clean up code, especially the Logger Backend itself, too messy now
- Should really test against exceptions and EXIT signals etc..
- LogLite is built with the way we're using (stackless)python in mind, so I'd like to improve how BEAM would fit into LogLite. Or maybe fork LogLite and fit it into BEAM.
- At least one defect I noticed during testing is that LogLite has problem handling Unicode, if a large message breaks in between a Unicode codepoint. Might need to change how LogLite decodes received data.

I hope ExLogLite can help programming and development in Elixir as that's what set me off to code it.
