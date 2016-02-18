---
layout: default
---

## rust_binary

```
rust_binary(name, srcs, deps, data, crate_features, rustc_flags)
```

### Attributes

<table class="params-table params-table-primary">
  <colgroup>
    <col class="col-param" />
    <col class="col-description" />
  </colgroup>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <p><code>Name, required</code></p>
        <p>A unique name for this rule.</p>
        <p>
          This name will also be used as the name of the binary crate built by
          this rule.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>srcs</code></td>
      <td>
        <p><code>List of labels, required</code></p>
        <p>List of Rust <code>.rs</code> source files used to build the
        binary.</p>
        <p>
          If <code>srcs</code> contains more than one file, then there must be
          a file either named <code>main.rs</code>. Otherwise,
          <code>crate_root</code> must be set to the source file that is the
          root of the crate to be passed to <code>rustc</code> to build this
          crate.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>crate_root</code></td>
      <td>
        <p><code>Label, optional</code></p>
        <p>
          The file that will be passed to <code>rustc</code> to be used for
          building this crate.
        </p>
        <p>
          If <code>crate_root</code> is not set, then this rule will look for
          a <code>main.rs</code> file or the single file in <code>srcs</code>
          if <code>srcs</code> contains only one file.
        </p>
      </td>
    </td>
    <tr>
      <td><code>deps</code></td>
      <td>
        <p><code>List of labels, optional</code></p>
        <p>List of other libraries to be linked to this library target.</p>
        <p>
          These must be <code>rust_library</code> targets.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>data</code></td>
      <td>
        <p><code>List of labels, optional</code></p>
        <p>List of files used by this rule at runtime.</p>
        <p>
          This attribute can be used to specify any data files that are embedded
          into the library, such as via the
          <a href="https://doc.rust-lang.org/std/macro.include_str!.html target="_blank"><code>include_str!</code></a>
          macro.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>crate_features</code></td>
      <td>
        <p><code>List of strings, optional</code></p>
        <p>List of features to enable for this crate.</p>
        <p>
          Features are defined in the code using the
          <code>#[cfg(feature = "foo")]</code> configuration option. The
          features listed here will be passed to <code>rustc</code> with
          <code>--cfg feature="${feature_name}"</code> flags.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>rustc_flags</code></td>
      <td>
        <p><code>List of strings, optional</code></p>
        <p>List of compiler flags passed to <code>rustc</code>.</p>
      </td>
    </tr>
  </tbody>
</table>

### Example

Suppose you have the following directory structure for a Rust project with a
library crate, `hello_lib`, and a binary crate, `hello_world` that uses the
`hello_lib` library:

```
[workspace]/
    WORKSPACE
    hello_lib/
        BUILD
        src/
            lib.rs
    hello_world/
        BUILD
        src/
            main.rs
```

`hello_lib/src/lib.rs`:

```rust
pub struct Greeter {
    greeting: String,
}

impl Greeter {
    pub fn new(greeting: &str) -> Greeter {
        Greeter { greeting: greeting.to_string(), }
    }

    pub fn greet(&self, thing: &str) {
        println!("{} {}", &self.greeting, thing);
    }
}
```

`hello_lib/BUILD`:

```python
package(default_visibility = ["//visibility:public"])

load("@bazel_tools//tools/build_rules/rust:rust.bzl", "rust_library")

rust_library(
    name = "hello_lib",
    srcs = ["src/lib.rs"],
)
```

`hello_world/src/main.rs`:

```rust
extern crate hello_lib;

fn main() {
    let hello = hello_lib::Greeter::new("Hello");
    hello.greet("world");
}
```

`hello_world/BUILD`:

```python
load("@bazel_tools//tools/build_rules/rust:rust.bzl", "rust_binary")

rust_binary(
    name = "hello_world",
    srcs = ["src/main.rs"],
    deps = ["//hello_lib"],
)
```

Build and run `hello_world`:

```
$ bazel run //hello_world
INFO: Found 1 target...
Target //examples/rust/hello_world:hello_world up-to-date:
  bazel-bin/examples/rust/hello_world/hello_world
INFO: Elapsed time: 1.308s, Critical Path: 1.22s

INFO: Running command line: bazel-bin/examples/rust/hello_world/hello_world
Hello world
```
