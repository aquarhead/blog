---
layout: post
title: Nice and ESI (Easy) `Task.async_stream`
cn: false
---

Today a colleague asked me whether I can help him figure out how to use `grequests` and Python 3 to concurrently fetch all [EVE](https://www.eveonline.com/)'s solar systems' security status via [ESI](https://esi.tech.ccp.is/latest/). After a few attempt, we found that Windows cannot handle more than 1024 sockets at the same time, and `grequests` is not clear on how to set a maximum concurrent connections. [^4]

But anyway I'm not a fan of Python things, so I told him I'm gonna write something in Elixir and provide him just the information he needs. Within 30 minutes or so I wrote the whole thing - including looking up a bunch of documentation and waiting for all those 8k queries (only twice though), I felt great the whole time, and I want to write down the entire process and explain how easy it is to do concurrent things with `Task.async_stream`.

So first I make clear the requirements:

- What I want: a [Map](https://hexdocs.pm/elixir/Map.html) of `system_id` -> `security_status`
- What I have: two ESI endpoints:
  - `/systems/` to get a list of `system_id`s
  - `/systems/{system_id}/` to get information of that system, which has its `security_status`

## Use `Tesla` to abstract API calls

[`Tesla`](https://hex.pm/packages/tesla) is a neat HTTP client library, which I encountered during the later stage of [card_labeler](https://github.com/aquarhead/card_labeler/commit/46c75c99dc2affd82d64419cf75452683b747c08). Using `plug` to introduce a series of middlewares just feels very clean yet very powerful. So coming to this case I immediately created an abstraction with it, let's look at the setup first:

```elixir
defmodule ESI.Client do
  use Tesla

  plug Tesla.Middleware.BaseUrl, "https://esi.tech.ccp.is/latest/universe"
  plug Tesla.Middleware.Headers, %{"User-Agent" => "eve_gamedesign"}
  plug Tesla.Middleware.JSON

  adapter Tesla.Adapter.Hackney

  # actual request functions...
end
```

There's really no magic here, the first two `plug`s setup the base url and headers for each request I'm going to make through this client, and the `JSON` middleware will encode/decode request/response bodies automatically. `Tesla` is very flexible, so you can use different underlying HTTP client, here I'm using [`hackney`](https://hex.pm/packages/hackney).

Then I defined a few meaningful requests we're going to make

```elixir
def get_systems do
  get("/systems/")
  |> Map.get(:body)
end

def get_system(system_id) do
  ss = get("/systems/#{system_id}/")
  |> Map.get(:body)
  |> Map.get("security_status")

  {system_id, ss}
end
```

`get_systems` will do a GET request to [https://esi.tech.ccp.is/latest/universe/systems/](https://esi.tech.ccp.is/latest/universe/systems/) (because we plugged `BaseUrl`).

When `Tesla` receives the response, it uses the `JSON` plug [^1] to decode its `body`, so we'll simply get a list of `system_id`s by calling `get_systems`.

`get_system(system_id)` is a similar story, but in this case, we only care about the security status of a system, so we extract that and only return the `system_id` with its `security_status`.

By returning in a `[{key, value}]` style [^2], we can turn it into a Map of `%{key => value}` using [`Enum.into`](https://hexdocs.pm/elixir/Enum.html#into/2)

```elixir
iex(1)> ESI.Client.get_systems |> Enum.take(5)
[30000001, 30000002, 30000003, 30000004, 30000005]
iex(2)> [ESI.Client.get_system(30000001)] |> Enum.into(%{})
%{30000001 => 0.8583240509033203}
```

## `Enum.map` -> `Task.async_stream`

If there aren't too many systems, I can simple use [`Enum.map`](https://hexdocs.pm/elixir/Enum.html#map/2) to turn a list of `system_id`s into pairs of `{system_id, security_status}` and then convert it into a Map. Like this:

```elixir
ESI.Client.get_systems()
|> Enum.map(fn sid -> ESI.Client.get_system(sid) end)
|> Enum.into(%{})
```

But there're 8000+ systems in EVE, so I want to make concurrent HTTP requests to utilize both our CPU and network's maximum capacity. This is where [`Task.async_stream`](https://hexdocs.pm/elixir/Task.html#async_stream/5) easily kicks in:

```elixir
ESI.Client.get_systems()
|> Task.async_stream(fn sid -> ESI.Client.get_system(sid) end)
|> Enum.map(fn {:ok, v} -> v end)
|> Enum.into(%{})
```

Notice how little I changed the sequential code to get a concurrent one (and with sliding window):

- I changed `Enum.map` to `Task.async_stream`: a [Stream](https://hexdocs.pm/elixir/Stream.html) is just a lazy enumerable (imagine an integer list from 1 to infinity), it only generates/evaluates/executes when and however much you ask it to, (so you ask 2 from the infinite list, it gives you `[1, 2]`, ask 2 more it gives you `[3, 4]` etc..). Now, instead of being mapped to each solar system's actual security status, the list becomes a list of *to-be-executed tasks* (`get_system`). Then, when I further apply any non-lazy functions, it actually starts executing.
- I then add `Enum.map(fn {:ok, v} -> v end)` to unwrap the result of those to-be-executed tasks, as the documentation says [^3]
  > "When streamed, each task will emit {:ok, value} upon successful.."

And that's it, that's *ALL* the change I need to convert a sequentially, eagerly executed series of [Enumerable](https://hexdocs.pm/elixir/Enumerable.html) operations to a concurrent one.

Note, that by default, for `Task.async_stream`

> The level of concurrency can be controlled via the `:max_concurrency` option and defaults to `System.schedulers_online/0`.

This means by default it only distribute the workload based on how many cores available, so I *think* by bumping the `:max_concurrency` to an even higher number the performance would be even better, because this problem should be network-bound, rather than CPU-bound.

Anyway, this is the first time I ever tried to write any `async_stream` code, and it feels awesome. And lovely.

:heart: :elixir:

[^1]: Which uses [Poison](https://hex.pm/packages/poison)
[^2]: Remember I got a list of `system_id`s
[^3]: Yeah, I didn't handle what happens if a task fails
[^4]: Maybe it can... BUT!
