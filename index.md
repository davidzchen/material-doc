---
layout: default
---

<h1 class="page-title">Setup</h1>

To use the Rust rules, add the following to your `WORKSPACE` file to add the
external repositories for the Rust toolchain:

```python
load("@bazel_tools//tools/build_rules/rust:rust.bzl", "rust_repositories")

rust_repositories()
```

## Roadmap

* Add `rust_toolchain` rule to make it easy to use a custom Rust toolchain.
* Add tool for taking `Cargo.toml` and generating a `WORKSPACE` file with
  workspace rules for pulling external dependencies.
* Improve expressiveness of features and support for [Cargo's feature
  groups](http://doc.crates.io/manifest.html#the-[features]-section).
* Add `cargo_crate` workspace rule for pulling crates from
  [Cargo](https://crates.io/).
