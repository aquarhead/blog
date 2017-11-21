---
layout: post
title: Autonomous `gen_statem`
cn: false
---

Nowadays, I usually write [Elixir](https://elixir-lang.org/) in my spare time. Not only that, actually whenever I encounter a problem at work, or have some random new ideas, I always think about how I can appraoch it in Elixir - or rather [OTP style](http://erlang.org/doc/design_principles/des_princ.html).

The more I try to design or architect things, the more often I find myself reaching out to [`gen_statem`](http://erlang.org/doc/design_principles/statem.html) (a successor to `gen_fsm`) instead of [`gen_server`](http://erlang.org/doc/design_principles/gen_server_concepts.html). A lot of things just naturally evolves over time between different states, and they should respond to things differently according to what state they are in.

The only downside is `gen_statem` does not have a wrapper in Elixir like [`GenServer`](https://hexdocs.pm/elixir/GenServer.html), so you can't `use GenServer` and import a bunch of default callbacks - doesn't really matter anyway. I gave this a real try in [a rewrite of card_labeler](https://github.com/aquarhead/card_labeler/commit/46c75c99dc2affd82d64419cf75452683b747c08), but before that I also spend a little time to figure out how I can make the state machine to make state transitions on its own. And this is my topic today.

## States and Events (Messages)

By default, a state machine can only make state transition when handling incoming events, but in my cases I want it to do something when _entering_ a state and directly transit itself to the next state when finished. For example the states I imagined for card_labeler include:

1. `preparing`: initialization, but because this reaches to external API, I shouldn't put it in `init/1`
2. `resting`: sleep for sometime, either because I don't want to be rate limited or I simply don't need to update that frequently
3. `tracking`: track issue changes using GitHub API
4. `updating`: update the project board accordingly using the tracked update

And it should transit between these states on its own, that's why I came up with this fancy name "autonomous state machine". I'm not really sure whether this totally makes sense or whatnot, but organizing different actions using different states makes my code a lot cleaner.

Once again I used [my Elixir playground](https://github.com/aquarhead/mashiro_no_asobiba) to test how I can achieve this. By reading the documentation closer, the key is to use [state enter calls](http://erlang.org/doc/design_principles/statem.html#id73392). This basically sends a state machine itself a message (`:enter`) when trasitting to a new state. To enable it just add it in the `callback_mode` setup, like:

`def callback_mode, do: [:state_functions, :state_enter]`

Note: by enabling it you need to handle the `:enter` message for all states.

## Example

You can checkout my full example [here](https://github.com/aquarhead/mashiro_no_asobiba/blob/master/lib/statem.exs) and by running it (`mix run --no-halt lib/statem.exs`) you should see logs like this:

```
19:51:06.138 [debug] inited

19:51:06.138 [debug] preparing

19:51:06.138 [debug] prepared

19:51:06.138 [debug] resting from preparing

19:51:06.141 [debug] tracking

19:51:06.141 [debug] tracked

19:51:06.141 [debug] updating

19:51:06.141 [debug] updated

19:51:06.141 [debug] resting from updating

19:51:11.142 [debug] tracking

19:51:11.142 [debug] tracked

19:51:11.142 [debug] updating

19:51:11.142 [debug] updated

19:51:11.142 [debug] resting from updating

19:51:16.142 [debug] tracking

19:51:16.142 [debug] tracked

19:51:16.142 [debug] updating

19:51:16.142 [debug] updated

19:51:16.142 [debug] resting from updating
```

## Closing

And that's all, I should have written this thing a long time ago. Anyway, I plan to write more small and simple posts like this whenever I discover something new in Elixir.
