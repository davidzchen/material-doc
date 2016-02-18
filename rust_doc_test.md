---
layout: default
---

## rust\_doc\_test

```python
rust_doc_test(name, dep)
```

### Attributes

<table class="params-table params-table-primary">
  <colgroup>
    <col class="col-param" />
    <col class="col-description" />
  </colgroup>
  <thead>
    <tr>
      <th>Attribute</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <code>Name, required</code>
        <p>A unique name for this rule.</p>
      </td>
    </tr>
    <tr>
      <td><code>dep</code></td>
      <td>
        <code>Label, required</code>
        <p>The label of the target to run documentation tests for.</p>
        <p>
          <code>rust_doc_test</code> can run documentation tests for the
          source files of <code>rust_library</code> or <code>rust_binary</code>
          targets.
        </p>
      </td>
    </tr>
  </tbody>
</table>

### Example

Suppose you have the following directory structure for a Rust library crate:

```
[workspace]/
    WORKSPACE
    hello_lib/
        BUILD
        src/
            lib.rs
```

To run [documentation tests][doc-test] for the `hello_lib` crate, define a
`rust_doc_test` target that depends on the `hello_lib` `rust_library` target:

[doc-test]: https://doc.rust-lang.org/book/documentation.html#documentation-as-tests

```python
package(default_visibility = ["//visibility:public"])

load("@bazel_tools//tools/build_rules/rust:rust.bzl", "rust_library", "rust_doc_test")

rust_library(
    name = "hello_lib",
    srcs = ["src/lib.rs"],
)

rust_doc_test(
    name = "hello_lib_doc_test",
    dep = ":hello_lib",
)
```

Running `bazel test //hello_lib:hello_lib_doc_test` will run all documentation
tests for the `hello_lib` library crate.
