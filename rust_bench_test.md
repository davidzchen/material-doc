---
layout: default
---

## rust\_bench\_test

```python
rust_bench_test(name, srcs, deps, data, crate_features, rustc_flags)
```

**Warning**: This rule is currently experimental. [Rust Benchmark
tests][rust-bench] require the `Bencher` interface in the unstable `libtest`
crate, which is behind the `test` unstable feature gate. As a result, using
this rule would require using a nightly binary release of Rust. A
`rust_toolchain` rule will be added in the [near future](#roadmap) to make it
easy to use a custom Rust toolchain, such as a nightly release.

[rust-bench]: https://doc.rust-lang.org/book/benchmark-tests.html

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
          This name will also be used as the name of the binary test crate
          built by this rule.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>srcs</code></td>
      <td>
        <p><code>List of labels, required</code></p>
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
        <p><code>Label, optional</code></p>
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
        <p><code>List of labels, optional</code></p>
        <p>List of other libraries to be linked to this test target.</p>
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
library crate, `fibonacci` with benchmarks under the `benches/` directory:

```
[workspace]/
    WORKSPACE
    fibonacci/
        BUILD
        src/
            lib.rs
        benches/
            fibonacci_bench.rs
```

`fibonacci/src/lib.rs`:

```rust
pub fn fibonacci(n: u64) -> u64 {
    if n < 2 {
        return n;
    }
    let mut n1: u64 = 0;
    let mut n2: u64 = 1;
    for _ in 1..n {
        let sum = n1 + n2;
        n1 = n2;
        n2 = sum;
    }
    n2
}
```

`fibonacci/benches/fibonacci_bench.rs`:

```rust
#![feature(test)]

extern crate test;
extern crate fibonacci;

use test::Bencher;

#[bench]
fn bench_fibonacci(b: &mut Bencher) {
    b.iter(|| fibonacci::fibonacci(40));
}
```

To build the benchmark test, simply add a `rust_bench_test` target:

`fibonacci/BUILD`:

```python
package(default_visibility = ["//visibility:public"])

load("@bazel_tools//tools/build_rules/rust:rust.bzl", "rust_library", "rust_bench_test")

rust_library(
    name = "fibonacci",
    srcs = ["src/lib.rs"],
)

rust_bench_test(
    name = "fibonacci_bench",
    srcs = ["benches/fibonacci_bench.rs"],
    deps = [":fibonacci"],
)
```

Run the benchmark test using: `bazel build //fibonacci:fibonacci_bench`.
