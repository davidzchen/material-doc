---
layout: default
---

## rust\_repositories

```python
rust_repositories()
```

Add this rule to your `WORKSPACE` file to add the external repositories used by
the Rust rules.

## Example

```python
load("@bazel_tools//tools/build_rules/rust:rust.bzl", "rust_repositories")

rust_repositories()
```
