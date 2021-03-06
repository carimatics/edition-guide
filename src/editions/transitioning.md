# Transitioning your code to a new edition

New editions might change the way you write Rust -- they add new syntax,
language, and library features but also remove features. For example,
`async`/`await` are keywords in Rust 2018, but not Rust 2015. Despite this
it's our intention that the migration to new editions is as smooth an experience
as possible. It's considered a bug if it's difficult to upgrade your crate to a
new edition. If you have a difficult time then a bug should be filed with Rust
itself.

Transitioning between editions is built around compiler lints. Fundamentally,
the process works like this:

* Turn on lints to indicate where code is incompatible with a new edition
* Get your code compiling with no warnings.
* Opt in to the new edition, the code should compile.
* Optionally, enable lints about *idiomatic* code in the new edition.

Luckily, we've been working on Cargo to help assist with this process,
culminating in a new built-in subcommand `cargo fix`. It can take suggestions
from the compiler and automatically re-write your code to comply with new
features and idioms, drastically reducing the number of warnings you need to fix
manually!

> `cargo fix` is still quite young, and very much a work in development. But it
> works for the basics! We're working hard on making it better and more robust,
> but please bear with us for now.

## The preview period

Before an edition is released, it will have a "preview" phase which lets you
try out the new edition in nightly Rust before its release. Currently Rust 2018
is in its preview phase and because of this, there's an extra step you need to
take to opt in.  Add this feature flag to your `lib.rs` or `main.rs`:

```rust
#![feature(rust_2018_preview)]
```

This will enable the unstable features listed in the [feature status][status]
page. Note that some features require a miniumum of Rust 2018 and these features will
require a Cargo.toml change to enable (described in the sections below). Also
note that during the time the preview is available, we may continue to add/enable
new features with this flag!

[status]: 2018/status.html

## Fix edition compatibility warnings

Next up is to enable compiler warnings about code which is incompatible with the
new 2018 edition. This is where the handy `cargo fix` tool comes into the
picture. To enable the compatibility lints for your project you run:

```shell
$ cargo +nightly fix --prepare-for 2018 --all-targets --all-features
```

This will instruct Cargo to compile all targets in your project (libraries,
binaries, tests, etc.) while enabling all Cargo features and prepare them for
the 2018 edition. Cargo will likely automatically fix a number of files,
informing you as it goes along.  Note that this does not enable any new Rust
2018 features; it only makes sure your code is compatible with Rust 2018.

If Cargo can't automatically fix everything it'll print out the remaining
warnings. Continue to run the above command until all warnings have been solved.

You can explore more about the `cargo fix` command with:

```shell
$ cargo +nightly fix --help
```

## Switch to the next edition

Once you're happy with those changes, it's time to use the new edition.
Add this to your `Cargo.toml`:

```toml
cargo-features = ["edition"]

[package]
edition = '2018'
```

That `cargo-features` line should go at the very top; and `edition` goes into
the `[package]` section. As mentioned above, right now this is a nightly-only
feature of Cargo, so you need to enable it for things to work.

At this point, your project should compile with a regular old `cargo +nightly
build`. If it does not, this is a bug! Please [file an issue][issue].

[issue]: https://github.com/rust-lang/rust/issues/new

## Writing idiomatic code in a new edition

Your crate has now entered the 2018 edition of Rust, congrats! Recall though
that Editions in Rust signify a shift in idioms over time. While much old
code will continue to compile it might be written with different idioms today.

An optional next step you can take is to update your code to be idiomatic within
the new edition. This is done with a different set of "idiom lints". To enable
these lints add this to your `lib.rs` or `main.rs`:

```rust
#![warn(rust_2018_idioms)]
```

Note that since you already switched to the new edition in the previous section,
you should no longer pass the `--prepare-for 2018` flag to `cargo fix` because
you are now already on the next edition! Execute:

```shell
$ cargo +nightly fix
```

As before Cargo will automatically fix as much as it can, but you may also need
to fix some warnings manually. Once all warnings have been solved you're not
only compiling with the 2018 edition but you're also already writing idiomatic
2018 code!
