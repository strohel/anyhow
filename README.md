Anyhow&ensp;¯\\\_(ツ)\_/¯
=========================

[![Build Status](https://api.travis-ci.com/dtolnay/anyhow.svg?branch=master)](https://travis-ci.com/dtolnay/anyhow)
[![Latest Version](https://img.shields.io/crates/v/anyhow.svg)](https://crates.io/crates/anyhow)
[![Rust Documentation](https://img.shields.io/badge/api-rustdoc-blue.svg)](https://docs.rs/anyhow)

This library provides [`anyhow::Error`][Error], a trait object based error type
for easy idiomatic error handling in Rust applications.

[Error]: https://docs.rs/anyhow/1.0/anyhow/struct.Error.html

```toml
[dependencies]
anyhow = "1.0"
```

*Compiler support: requires rustc 1.34+*

<br>

## Details

- Use `Result<T, anyhow::Error>`, or equivalently `anyhow::Result<T>`, as the
  return type of any fallible function.

  Within the function, use `?` to easily propagate any error that implements the
  `std::error::Error` trait.

  ```rust
  use anyhow::Result;

  fn get_cluster_info() -> Result<ClusterMap> {
      let config = std::fs::read_to_string("cluster.json")?;
      let map: ClusterMap = serde_json::from_str(&config)?;
      Ok(map)
  }
  ```

- Attach context to help the person troubleshooting the error understand where
  things went wrong. A low-level error like "No such file or directory" can be
  annoying to debug without more context about what higher level step the
  application was in the middle of.

  ```rust
  use anyhow::{Context, Result};

  fn main() -> Result<()> {
      ...
      it.detach().context("failed to detach the important thing")?;

      let content = std::fs::read(path)
          .with_context(|| format!("failed to read instrs from {}", path))?;
      ...
  }
  ```

  ```console
  Error: failed to read instrs from ./path/to/instrs.jsox

  Caused by:
      No such file or directory (os error 2)
  ```

- Downcasting is supported and can be by value, by shared reference, or by
  mutable reference as needed.

  ```rust
  // If the error was caused by redaction, then return a
  // tombstone instead of the content.
  match root_cause.downcast_ref::<DataStoreError>() {
      Some(DataStoreError::Censored(_)) => Ok(Poll::Ready(REDACTED_CONTENT)),
      None => Err(error),
  }
  ```

- A backtrace is captured and printed with the error if the underlying error
  type does not already provide its own. In order to see backtraces, the
  `RUST_LIB_BACKTRACE=1` environment variable must be defined.

- Anyhow works with any error type that has an impl of `std::error::Error`,
  including ones defined in your crate. We do not bundle a `derive(Error)` macro
  but you can write the impls yourself or use a standalone macro like
  [thiserror].

  [thiserror]: https://github.com/dtolnay/thiserror

  ```rust
  use thiserror::Error;

  #[derive(Error, Debug)]
  pub enum FormatError {
      #[error("invalid header (expected {expected:?}, got {found:?})")]
      InvalidHeader {
          expected: String,
          found: String,
      },
      #[error("missing attribute: {0}")]
      MissingAttribute(String),
  }
  ```

- One-off error messages can be constructed using the `anyhow!` macro, which
  supports string interpolation and produces an `anyhow::Error`.

  ```rust
  return Err(anyhow!("missing attribute: {}", missing));
  ```

<br>

## Comparison to failure

The `anyhow::Error` type works something like `failure::Error`, but unlike
failure ours is built around the standard library's `std::error::Error` trait
rather than a separate trait `failure::Fail`. The standard library has adopted
the necessary improvements for this to be possible as part of [RFC 2504].

[RFC 2504]: https://github.com/rust-lang/rfcs/blob/master/text/2504-fix-error.md

<br>

## Acknowledgements

The implementation of the `anyhow::Error` type is originally forked from
`fehler::Exception` (https://github.com/withoutboats/fehler). This library
exposes it under the more standard `Error` / `Result` terminology rather than
the `throw!` / `#[throws]` / `Exception` language of exceptions.

<br>

#### License

<sup>
Licensed under either of <a href="LICENSE-APACHE">Apache License, Version
2.0</a> or <a href="LICENSE-MIT">MIT license</a> at your option.
</sup>

<br>

<sub>
Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this crate by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
</sub>
