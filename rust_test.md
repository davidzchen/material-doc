---
layout: default
---

## rust_test

```python
rust_test(name, srcs, deps, data, crate_features, rustc_flags)
```

### Attributes

<table class="table table-condensed table-bordered table-params">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <code>Name, required</code>
        <p>A unique name for this rule.</p>
        <p>
          This name will also be used as the name of the binary test crate
          built by this rule.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>srcs</code></td>
      <td>
        <code>List of labels, required</code>
        <p>List of Rust <code>.rs</code> source files used to build the
        library.</p>
        <p>
          If <code>srcs</code> contains more than one file, then there must be
          a file either named <code>lib.rs</code>. Otherwise,
          <code>crate_root</code> must be set to the source file that is the
          root of the crate to be passed to <code>rustc</code> to build this
          crate.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>crate_root</code></td>
      <td>
        <code>Label, optional</code>
        <p>
          The file that will be passed to <code>rustc</code> to be used for
          building this crate.
        </p>
        <p>
          If <code>crate_root</code> is not set, then this rule will look for
          a <code>lib.rs</code> file or the single file in <code>srcs</code>
          if <code>srcs</code> contains only one file.
        </p>
      </td>
    </td>
    <tr>
      <td><code>deps</code></td>
      <td>
        <code>List of labels, optional</code>
        <p>List of other libraries to be linked to this test target.</p>
        <p>
          These must be <code>rust_library</code> targets.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>data</code></td>
      <td>
        <code>List of labels, optional</code>
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
        <code>List of strings, optional</code>
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
        <code>List of strings, optional</code>
        <p>List of compiler flags passed to <code>rustc</code>.</p>
      </td>
    </tr>
  </tbody>
</table>

### Example

Suppose you have the following directory structure for a Rust library crate
with unit test code in the library sources:

```
[workspace]/
    WORKSPACE
    hello_lib/
        BUILD
        src/
            lib.rs
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

#[cfg(test)]
mod test {
    use super::Greeter;

    #[test]
    fn test_greeting() {
        let hello = Greeter::new("Hi");
        assert_eq!("Hi Rust", hello.greeting("Rust"));
    }
}
```

To build and run the tests, simply add a `rust_test` rule with no `srcs` and
only depends on the `hello_lib` `rust_library` target:

`hello_lib/BUILD`:

```python
package(default_visibility = ["//visibility:public"])

load("@bazel_tools//tools/build_rules/rust:rust.bzl", "rust_library", "rust_test")

rust_library(
    name = "hello_lib",
    srcs = ["src/lib.rs"],
)

rust_test(
    name = "hello_lib_test",
    deps = [":hello_lib"],
)
```

Run the test with `bazel build //hello_lib:hello_lib_test`.

### Example: `test` directory

Integration tests that live in the [`tests` directory][int-tests], they are
essentially built as separate crates. Suppose you have the following directory
structure where `greeting.rs` is an integration test for the `hello_lib`
library crate:

[int-tests]: http://doc.rust-lang.org/book/testing.html#the-tests-directory

```
[workspace]/
    WORKSPACE
    hello_lib/
        BUILD
        src/
            lib.rs
        tests/
            greeting.rs
```

`hello_lib/tests/greeting.rs`:

```rust
extern crate hello_lib;

use hello_lib;

#[test]
fn test_greeting() {
    let hello = greeter::Greeter::new("Hello");
    assert_eq!("Hello world", hello.greeting("world"));
}
```

To build the `greeting.rs` integration test, simply add a `rust_test` target
with `greeting.rs` in `srcs` and a dependency on the `hello_lib` target:

`hello_lib/BUILD`:

```python
package(default_visibility = ["//visibility:public"])

load("@bazel_tools//tools/build_rules/rust:rust.bzl", "rust_library", "rust_test")

rust_library(
    name = "hello_lib",
    srcs = ["src/lib.rs"],
)

rust_test(
    name = "greeting_test",
    srcs = ["tests/greeting.rs"],
    deps = [":hello_lib"],
)
```

Run the test with `bazel build //hello_lib:hello_lib_test`.
