<!-- markdownlint-disable MD013 -->

# More about Cargo and Crates.io

<!-- prettier-ignore-start -->

<!--toc:start-->
- [More about Cargo and Crates.io](#more-about-cargo-and-cratesio)
  - [Customizing Builds with Release Profiles](#customizing-builds-with-release-profiles)
  - [Publishing a Crate to Crates.io](#publishing-a-crate-to-cratesio)
    - [Making Useful Documentation Comments](#making-useful-documentation-comments)
      - [Commonly Used Sections](#commonly-used-sections)
      - [Documentation Comments as Tests](#documentation-comments-as-tests)
      - [Commenting Contained Items](#commenting-contained-items)
    - [Exporting a Convenient Public API with `pub use`](#exporting-a-convenient-public-api-with-pub-use)
    - [Setting Up a Crates.io Account](#setting-up-a-cratesio-account)
    - [Adding Metadata to a New Crate](#adding-metadata-to-a-new-crate)
    - [Publishing to Crates.io](#publishing-to-cratesio)
    - [Publishing a New Version of an Existing Crate](#publishing-a-new-version-of-an-existing-crate)
    - [Deprecating Versions from Crates.io with `cargo yank`](#deprecating-versions-from-cratesio-with-cargo-yank)
  - [Cargo Workspaces](#cargo-workspaces)
    - [Creating a Workspace](#creating-a-workspace)
    - [Creating Other Packages in the Workspace](#creating-other-packages-in-the-workspace)
      - [Depending on an External Package in a Workspace](#depending-on-an-external-package-in-a-workspace)
      - [Adding a Test to a Workspace](#adding-a-test-to-a-workspace)
  - [Installing Binaries with `cargo install`](#installing-binaries-with-cargo-install)
  - [Extending Cargo with Custom Commands](#extending-cargo-with-custom-commands)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Customizing Builds with Release Profiles

In Rust, _release profiles_ are predefined and customizable profiles with different configurations that allow a programmer to have more control over various options for compiling code. Each profile is configured independently of the others.

Cargo has two main profiles: `dev` profile Cargo uses when we run `cargo build` and `release` profile that Cargo uses when we run `cargo build --release`. `dev` profile is defined with good defaults for development, and `release` profile has good default for release builds.

```text
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
```

Cargo has default settings for each of the profiles that apply when we haven't explicitly added `[profile *]` sections in `Cargo.toml` file. Any added sections would override default settings. Below are default values of `opt-level` setting for `dev` and `release` profiles:

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

The `opt-level` controls the number of optimizations Rust will apply to our code, with a range of 0 to 3.

## Publishing a Crate to Crates.io

The crate registry at [crates.io](https://crates.io) distributes source code of various packages.

### Making Useful Documentation Comments

Rust has a kind of comment for documentation called _documentation comment_, that will generate HTML documentation. The HTML displays documentation contents for public API items intended for others using our library.

Documentation comments use three slashes, `///`, and supports Markdown notation. Place documentation comments just before the item being documented. Below is an example:

````rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
````

The HTML documentation can be generated from documentation comments by running `cargo doc`, which runs the `rustdoc` tool distributed with Rust and puts the generated HTML documentation in the _target/doc_ directory.

For convenience, running `cargo doc --open` builds the HTML for our current crate's documentation as well as documentation for all of crate's dependencies.

#### Commonly Used Sections

In the code above, we used the `# Examples` Markdown heading to create a section in the HTML with the `Examples` title. Here are some other sections commonly used in documentation:

- Panics: the scenarios in which the function being documented could panic.
- Errors: if the function returns a `Result`, describing what kind of errors might occur and what conditions might cause those errors to be returned can be helpful to callers.
- Safety: If the function is `unsafe` to call, there should be a section explaining why the function is unsafe and covering the invariants that the function expects callers to uphold.

#### Documentation Comments as Tests

Running `cargo test` will run code examples in documentation as tests.

#### Commenting Contained Items

Use comment style `//!` to add comment that describes the file/item that contains the comment rather than the item following the comment.

### Exporting a Convenient Public API with `pub use`

Sometimes when developing a crate, the structure may be nested and complicated which is not convenient for crate user to import. We can improve this by re-exporting items using `pub use`. Re-exporting takes a public item in one location and makes it public in another location, as if it were defined in the other location instead.

For example, if we have an `Art` library defined as follows:

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
    }
}
```

The generated documentation will have `kinds`, `utils` listed under "Modules" section. Here's an example showing how to import items:

```rust
use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

Notice how we need to import items from their namespace? To remove internal structure from public API, we can re-export the items at top level.

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

In generated documentation, the 3 re-exported items will be listed under `Re-exports` section. Those items can be imported directly as follows:

```rust
use art::mix;
use art::PrimaryColor;

fn main() {
    // --snip--
}
```

Another common use of `pub use` is to re-export definitions of a dependency in the current crate to make that crate's definitions part of our crate's public API.

### Setting Up a Crates.io Account

Before we can publish crates, we need to create account on [crates.io](https://crates.io) and get an API token. To get API token, log in at [crates.io](crates.io) using GitHub account. Once logged in, visit [https://crates.io/me/](https://crates.io/me/) and retrieve API key.

To login in command line, run `cargo login` and paste in API key.

```text
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

Cargo then stores it locally in `~/.cargo/credentials`. Note this API token is a _secret_, and it should not be shared. If the token is leaked, revoke it and generate a new token from [crates.io](crates.io).

### Adding Metadata to a New Crate

Before publishing package to crates, we need to add some required metadata in the `[package]` section in _Cargo.toml_ file:

- name: unique create name that is not taken on [crates.io](https://crates.io)
- description: one to two sentences because it will appear with our crate in search results.
- license: _license identifier value_. The [Linux Foundation’s Software Package Data Exchange (SPDX)](https://spdx.org/licenses/) lists the identifiers. If a license does not appear in the SPDX, we need to place text of the license in a file, and ispecify the file name in `license-file`.

Some other optional metadata, such as documentation, homepage, repository, edition, version, etc.

To publish, run `cargo publish`

Example _Cargo.toml_

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

### Publishing to Crates.io

A publish is permanent as major goal of [crates.io](https://crates.io) is to act as a permanent archive of code. The version can never be overwritten or deleted, and there is no limit to the number of crate versions we can publish.

### Publishing a New Version of an Existing Crate

To release a new version of the crate, change the `version` value in package metadata in _Cargo.toml_, and republish by running `cargo publish`. Use the [Semantic Versioning Rules](https://semver.org) to decide the appropriate version.

### Deprecating Versions from Crates.io with `cargo yank`

To deprecate a version of a crate, run `cargo yank`. Deprecating a version means all dependant projects will not break, and future dependant projects will not use the deprecated version. Deprecating does not delete code; if a secret is uploaded accidentally, those secret must be reset/revoked.

Example:

```text
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

To undo a yank:

```text
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

## Cargo Workspaces

### Creating a Workspace

A _workspace_ is a set of packages that share the same _Cargo.lock_ and output directory.

There are many ways to structure a workspace, but we'll demonstrate a common way to organize a workspace.

For our example, we'll create a workspace with 1 binary and 2 libraries. The binary provides the main functionality and depends on the two libraries. One library provides an `add_one` function and the other library provides an `add_two` function.

To create a workspace, first create workspace directory.

```sh
mkdir add

cd add

```

In _add_ directory, create _Cargo.toml_ to configure the workspace, which starts with `[workspace]` section that will allow us to add members to the workspace by specifying the path to the package with our binary crate. In the example, it's _adder_.

```toml
[workspace]

members = [
    "adder",
]
```

Next, create the adder library by running `cargo new adder`, then run `cargo build` to build the workspace. The directory should have structure as follows:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

The workspace has 1 _target_ directory at the top level that contains compiled artifacts. Even if we run `cargo build` from inside _adder_ directory, the compiled artifacts would still end up in _add/target_ in workspace directory. By sharing one _target_ directory for all packages in a workspace, the crates can avoid unnecessary rebuilding.

### Creating Other Packages in the Workspace

Add new package directories in workspace _Cargo.toml_

```toml
[workspace]

members = [
    "adder",
    "add_one",
    "add_two",
]
```

Generate the packages:

```text
$ cargo new add_one --lib
     Created library `add_one` package
$ cargo new add_two --lib
     Created library `add_two` package
```

The workspace structure should looks as follows:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── add_two
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Add `add_one` function in _add_one/src/lib.rs_.

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

And `add_two` function in _add_two/src/lib.rs_.

```rust
pub fn add_two(x: i32) -> i32 {
    x + 2
}
```

To use the library functions in _adder_ crate, specify dependencies in _adder/Cargo.toml_.

```toml
[dependencies]
add_one = { path = "../add_one" }
add_two = { path = "../add_two" }
```

Now we can use library functions in _adder_ crate. Modify file _adder/src/main.rs_:

```rust
use add_one;
use add_two;

fn main() {
    let num = 10;
    println!("Hello, world! {num} plus one is {}!", add_one::add_one(num));
    let num2 = 11;
    println!("Hello, world! {num} plus two is {}!", add_two::add_two(num2));
}

```

Rebuild workspace by running `cargo build`. To run binary crate from workspace directory, use `-p` to specify binary crate. For instance, `cargo run -p adder`.

#### Depending on an External Package in a Workspace

Workspace has only one _Cargo.lock_ at the top level, rather than in each crate's directory. This ensures all crates are using the same version of all dependencies.

To add external package into a crate, add it to `[dependencies]` section in _Cargo.toml_ in individual crate. For example, if we want to add `rand` crate to _add_one_ library, modify _add_one/Cargo.toml_:

```toml
[dependencies]
rand = "0.8.5"
```

The `rand` crate can now be used in _adder_one_. If we want to use `rand` in _adder_, we would need to add it to _adder/Cargo.toml_.

Building _adder_ package will add `rand` to list of dependencies for `adder` in _Cargo.lock_, but no additional copies of `rand` will be downloaded. Cargo ensures every crate in every package in the workspace uses same version of `rand` as long as the version specified are compatible. If crates in the workspace specify incompatible versions of the same dependency, Cargo will resolve each of them, but try to resolve as few versions as possible.

#### Adding a Test to a Workspace

Running `cargo test` in workspace directory runs tests in all crates. To specify which crate to run tests on, use `-p` flag; for instance, `cargo run -p add_one`.

If we want to publish crates in the workspace to [crates.io](https://crates.io), each crate in the workspace will need to be published separately; for instance, `cargo publish -p add_one`

## Installing Binaries with `cargo install`

Run `cargo install` to install and use binary crates locally. Note that only packages with binary targets can be installed. A _binary target_ is the runnable program that is created if the crate has _src/main.rs_ file or another file specified as binary, as opposed to library target that isn't runnable on its own.

The installed binaries are stored in _\$HOME/.cargo/bin_ by default.

Example:

```text
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v13.0.0
  Downloaded 1 crate (243.3 KB) in 0.88s
  Installing ripgrep v13.0.0
--snip--
   Compiling ripgrep v13.0.0
    Finished release [optimized + debuginfo] target(s) in 3m 10s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v13.0.0` (executable `rg`)
```

If _\$HOME/.cargo/bin_ is in `$PATH`, we can now run`rg` in command line to start using `ripgrep`.

## Extending Cargo with Custom Commands

If a binary in `$PATH` is named `cargo-something`, you can run it as if it was a Cargo subcommand by running `cargo something`. Custom commands like this are also listed when you run `cargo --list`.
