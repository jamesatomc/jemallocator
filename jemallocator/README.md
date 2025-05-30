# tikv-jemallocator

[![ci]][github actions] [![Latest Version]][crates.io] [![docs]][docs.rs]

This project is the successor of [jemallocator](https://github.com/gnzlbg/jemallocator).

The project is also published as `jemallocator` for historical reasons. The two crates are the same except names. For new projects, it's recommended to use `tikv-xxx` versions instead.

> Links against `jemalloc` and provides a `Jemalloc` unit type that implements
> the allocator APIs and can be set as the `#[global_allocator]`

## Overview

The `jemalloc` support ecosystem consists of the following crates:

* `tikv-jemalloc-sys`: builds and links against `jemalloc` exposing raw C bindings to it.
  * On MSVC targets, it provides a stub implementation that forwards to the system allocator to avoid linking conflicts.
* `tikv-jemallocator`: provides the `Jemalloc` type which implements the
  `GlobalAlloc` and `Alloc` traits. 
* `tikv-jemalloc-ctl`: high-level wrapper over `jemalloc`'s control and introspection
  APIs (the `mallctl*()` family of functions and the _MALLCTL NAMESPACE_)'

## Documentation

* [Latest release (docs.rs)][docs.rs]

To use `tikv-jemallocator` add it as a dependency:

```toml
# Cargo.toml
[dependencies]
tikv-jemallocator = { git = "https://github.com/jamesatomc/jemallocator.git", features = [
    "unprefixed_malloc_on_supported_platforms",
    "profiling",
] }
```

To set `tikv_jemallocator::Jemalloc` as the global allocator add this to your project:

```rust
// main.rs
use tikv_jemallocator::Jemalloc;

#[global_allocator]
static GLOBAL: Jemalloc = Jemalloc;
```

And that's it! Once you've defined this `static` then jemalloc will be used for
all allocations requested by Rust code in the same program. On MSVC targets, the system allocator will be used instead to avoid linking conflicts.

## Avoiding Link Conflicts

When using this crate with other libraries that also link to jemalloc (such as RocksDB with the jemalloc feature enabled), you might encounter linking conflicts on some platforms. 

On MSVC targets, `tikv-jemalloc-sys` automatically uses a stub implementation that forwards to the system allocator and doesn't actually link to jemalloc, avoiding these conflicts.

If you're still experiencing linking conflicts, you may need to:
1. Disable jemalloc in one of the dependencies (e.g., use RocksDB without the jemalloc feature)
2. Ensure you're using a consistent version of jemalloc throughout your dependency tree

## Platform support

The following table describes the supported platforms: 

* `build`: does the library compile for the target?
* `run`: do `tikv-jemallocator` and `tikv-jemalloc-sys` tests pass on the target?
* `jemalloc`: do `tikv-jemalloc`'s tests pass on the target?

Tier 1 targets are tested on all Rust channels (stable, beta, and nightly). All
other targets are only tested on Rust nightly.

| Linux targets:                      | build     | run     | jemalloc     |
|-------------------------------------|-----------|---------|--------------|
| `aarch64-unknown-linux-gnu`         | ✓         | ✓       | ✗            |
| `powerpc64le-unknown-linux-gnu`     | ✓         | ✓       | ✗            |
| `x86_64-unknown-linux-gnu` (tier 1) | ✓         | ✓       | ✓            |
| **MacOSX targets:**                 | **build** | **run** | **jemalloc** |
| `aarch64-apple-darwin`              | ✓         | ✓       | ✗            |
| **Windows targets:**                | **build** | **run** | **jemalloc** |
| `x86_64-pc-windows-msvc`            | ✓         | ✓       | Note 1       |

Note 1: Refers to jemalloc's own comprehensive test suite. Basic allocator functionality provided by `tikv-jemallocator` is expected to work.

## Features

This crate provides following cargo feature flags:

* `alloc_trait` When the `alloc_trait` feature of this crate is enabled, it also implements the `Alloc` trait, allowing usage in collections.

* `default` feature is `background_threads_runtime_support`.

* The `tikv-jemallocator` crate re-exports the [features of the `tikv-jemalloc-sys`
dependency](https://github.com/tikv/jemallocator/blob/master/jemalloc-sys/README.md#features).

## License

This project is licensed under either of

 * Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or
   http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or
   http://opensource.org/licenses/MIT)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in `tikv-jemallocator` by you, as defined in the Apache-2.0 license,
shall be dual licensed as above, without any additional terms or conditions.

[Latest Version]: https://img.shields.io/crates/v/tikv-jemallocator.svg
[crates.io]: https://crates.io/crates/tikv-jemallocator
[docs]: https://docs.rs/tikv-jemallocator/badge.svg
[docs.rs]: https://docs.rs/tikv-jemallocator/
[ci]: https://github.com/tikv/jemallocator/actions/workflows/main.yml/badge.svg
[github actions]: https://github.com/tikv/jemallocator/actions
