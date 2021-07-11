---
layout: post
title: "Fully Automated Rust Code Generation for Large Protobuf/gRPC Repos"
---

TL; DR: add [this `build.rs`](https://gist.github.com/aquarhead/69092f21347353981357909bea7765e4#file-build-rs) and put `include!(concat!(env!("OUT_DIR"), "/protos.rs"));` somewhere in `lib.rs`.

## Preface

Recently, I've been playing with the protobuf/gRPC ecosystem in Rust, building a gRPC client against a message bus service at work. Through [lib.rs](https://lib.rs/) I found the most popular stack in Rust - `tonic` and `prost`, and overall they are extremely good foundation and very easy to use, with various examples showcasing all use cases I need.

However, one major pain point is the initial code generation and importing the generated code into a Rust library. My target service's protobuf and gRPC definitions are stored in a central repo and spread across many many files and multiple levels of directories. `tonic-build` - which is powered by `prost-build` - is currently oriented around code generation from a defined list of source `.proto` files, this works fine if an application does not need to interact with too many definitions in different files, but it clearly doesn't scale (our repo already contains several hundreds of proto files).

So, without support such as [the glob pattern](https://github.com/tokio-rs/prost/issues/469), I spent a few hours to figure out a solution (basically `build.rs`) that automatically compiles all the proto definitions, and also organize them correctly and make it easy to import. There are also several caveats that I encountered so you don't have to discover them again.

As a disclaimer, the code in this post is only tested for my case, though I believe it should only require minimal change to adapt to many other cases. I'm also not sure if there exists built-in support or other prior work, which could be better than what I have here. Let me know!

## The Basics

Let's start with a quick look at the basic API. Following various [tutorial](https://github.com/hyperium/tonic/blob/master/examples/helloworld-tutorial.md) and [documentations](https://docs.rs/tonic-build/0.5.0/tonic_build/index.html) it basically works as follows:

With some `.proto` files:

```protobuf
syntax = "proto3";

package parent.example;

message Example {}
```

Compile them through `build.rs`:

```rust
tonic_build::configure()
  .build_server(false)
  .compile(&["base/parent/example.proto"], &["base"])?;
```

Import the generated code in `lib.rs`:

```rust
pub mod parent {
  tonic::include_proto!("parent");

  pub mod example {
    tonic::include_proto!("parent.example");
  }
}
```

And finally the proto structs can be used:

```rust
let e = project::parent::example::Example {};
```

It's clear that we not only want to automate the code generation in `build.rs`, but also all those `include_proto!` calls in `lib.rs`. It's worth noting that the layers of `pub mod` are also crucial, otherwise inter-package references won't work, all the more reason to automate this part.

## The Generation

It's easy to expand the basic example to _compile_ all proto files, all we need is to recursively find and collect all the `.proto` files, then finally feed them to the compiler. It's possible to roll your own iterator, but I decided to just use `walkdir` (add it to `[build-dependencies]` in `Cargo.toml`).

There isn't much "magic" involved so I won't show the code here, but there's a caveat you may encounter - all proto files need to declare [the `pacakge` specifier]((https://developers.google.com/protocol-buffers/docs/proto3#packages)) as elaborated in [the README of `prost`](https://github.com/tokio-rs/prost#packages). This is not a requirement for example [in Elixir](https://github.com/elixir-protobuf/protobuf) which I've used extensively. Fortunately in my case, out of the hundreds of proto files, only one of them is missing `package`, it's likely a neglect in the early days and we now added linter rule to avoid this in the future.

## The Include

While finding all the files to _compile_ is rather easy, building the correct module hierarchy turns out to be quite involved. I wasted some time initially thinking that it's based on the file path, but eventually realized it's based on the `package` hierarchy. This requires reading the file _content_ [^1] and some parsing, we use formatters strictly which alleviates some issue, but it's still tricky for example I need to purge the comments.

[^1]: I do think this would be a lot cleaner if integrated inside `prost-build`

Multiple files can combine into a single `package` so a `HashSet` is used to avoid duplication. A simple `Vec` is used to collect all files.

{% raw %}
```rust
use std::{collections::HashSet, env, fmt::Write, fs, path::Path};
use walkdir::WalkDir;

type Res = Result<(), Box<dyn std::error::Error>>;

fn main() -> Res {
  let mut protos = vec![];
  let mut pkgs = HashSet::new();

  for entry in WalkDir::new("ROOT DIR")
    .into_iter()
    .map(|e| e.unwrap())
    .filter(|e| {
      e.path()
        .extension()
        .map_or(false, |ext| ext.to_str().unwrap() == "proto")
    })
  {
    let path = entry.path();
    protos.push(path.to_owned());

    let content = fs::read_to_string(&path).unwrap();
    let pkg = content
      .lines()
      .find(|line| line.starts_with("package "))
      .unwrap()
      // remove comment
      .split("//")
      .next()
      .unwrap()
      // extract package
      .trim()
      .trim_start_matches("package ")
      .trim_end_matches(";");

    pkgs.insert(pkg.to_string());
  }

  tonic_build::configure()
    .build_server(false)
    .compile(&protos, &[Path::new("INCLUDE DIR").into()])?;

  write_protos_rs(pkgs)?;

  println!("cargo:rerun-if-changed=REPO DIR");

  Ok(())
}
```
{% endraw %}

With all the `packages` specifiers collected, next is to figure out how to derive the hierarchy from it and write the `pub mod` accordingly. I initially thought about using a tree but eventually landed on a solution based on a conceptual "stack", with all packages sorted, it's the same as a depth-first tree traversal.

The stack contains "segments" of a `package` path, while iterating it first pops the stack to the "common ancestor" then pushs whatever segments left into the stack. With each push we open a module and with each pop we close one. This repeats for all packages and finally pop all the segments left to properly close all modules.

For example, when generating for `parent.example` the stack should contain `["parent", "example"]`, then for the next package `parent.another` the stack first pop to only `["parent"]`, closing `pub mod example {`, next push `"another"` into the stack and open the module `pub mod another {`, finally fill in the proper include macro.

This is the `write_protos_rs` function:

{% raw %}
```rust
fn write_protos_rs(pkgs: HashSet<String>) -> Res {
  let ref mut protos_rs = String::new();

  let mut packages: Vec<String> = pkgs.into_iter().collect();
  packages.sort();

  let mut path_stack: Vec<String> = vec![];

  for pkg in packages {
    // find common ancestor
    let pop_to = pkg
      .split(".")
      .map(|seg| map_keyword(seg))
      .enumerate()
      .position(|(idx, pkg_seg)| {
        path_stack
          .get(idx)
          .map_or(true, |stack_seg| stack_seg != &pkg_seg)
      })
      .unwrap_or(0);

    // pop stack
    while path_stack.len() > pop_to {
      path_stack.pop();
      writeln!(protos_rs, "}}")?;
    }

    // now push stack
    for seg in pkg.split(".").skip(pop_to).map(|seg| map_keyword(seg)) {
      writeln!(protos_rs, "pub mod {} {{", &seg)?;
      path_stack.push(seg);
    }

    // write include_proto! inside module
    writeln!(
      protos_rs,
      "tonic::include_proto!(\"{}\");",
      path_stack.join(".")
    )?;
  }

  // pop all stack
  while path_stack.len() > 0 {
    path_stack.pop();
    writeln!(protos_rs, "}}").unwrap();
  }

  fs::write(format!("{}/protos.rs", env::var("OUT_DIR")?), protos_rs)?;

  Ok(())
}
```
{% endraw %}

The function `map_keyword()` is necessary because `prost` transforms file names and path components that conflict with Rust keywords such as `type`, this applies to both the include path as well as the module name. I simply copied over [the code from `prost-build`](https://github.com/tokio-rs/prost/blob/bb986cd78fd03b40f16845ab46865cd3be9a9f6c/prost-build/src/ident.rs#L12-L29).

There is another caveat at the time of writing - `oneof` field can conflict with embedded message names, I [opened an issue in `prost`](https://github.com/tokio-rs/prost/issues/505) and I'm [working on a PR](https://github.com/tokio-rs/prost/pull/506). There is only one case in the repo I'm working against, and for my purpose I simply ignored (deleted) those protos, but I hope to get a working solution merged soon.

## The Conclusion

The full `build.rs` file [can be found here](https://gist.github.com/aquarhead/69092f21347353981357909bea7765e4) and with it all the generated code and modules can be imported by a single line in `lib.rs`:

```rust
include!(concat!(env!("OUT_DIR"), "/protos.rs"));
```

The generated code can also be segregated to a child module to avoid mixing with other code:

```rust
pub mod protos {
  include!(concat!(env!("OUT_DIR"), "/protos.rs"));
}
```

And that's it! Now all messages and services are automatically generated, imported and ready for use. Hope this post is useful for those with similiar use case, but let me know if there are better solutions!
