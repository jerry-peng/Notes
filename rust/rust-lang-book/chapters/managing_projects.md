<!-- markdownlint-disable MD013 -->

# Managing Growing Projects

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Managing Growing Projects](#managing-growing-projects)
  - [Packages and Crates](#packages-and-crates)
  - [Defining Modules to Control Scope and Privacy](#defining-modules-to-control-scope-and-privacy)
    - [Create new library](#create-new-library)
  - [Paths for Referring to an Item in the Module Tree](#paths-for-referring-to-an-item-in-the-module-tree)
    - [Exposing Paths with the `pub` Keyword](#exposing-paths-with-the-pub-keyword)
    - [Starting Relative Paths with `super`](#starting-relative-paths-with-super)
    - [Making Structs and Enums Public](#making-structs-and-enums-public)
  - [Bring Paths Into Scope with `use` Keyword](#bring-paths-into-scope-with-use-keyword)
    - [Idiomatic `use`](#idiomatic-use)
    - [Providing New Names with `as` Keyword](#providing-new-names-with-as-keyword)
    - [Re-exporting Names with `pub use`](#re-exporting-names-with-pub-use)
    - [Using External Packages](#using-external-packages)
    - [Using Nested Paths to Clean Up Large `use` Lists](#using-nested-paths-to-clean-up-large-use-lists)
    - [The Glob Operator](#the-glob-operator)
  - [Separating Modules into Different Files](#separating-modules-into-different-files)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Packages and Crates

Create new package

```text
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

## Defining Modules to Control Scope and Privacy

_Modules_ let us organize code within a crate into groups. Modules also controls the privacy of items (public/private).

### Create new library

```sh
cargo new --lib restaurant
```

```rust
// src/lib.rs
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}
```

`src/main.rs` and `src/lib.rs` are called crate roots, because contents of either files form a module named `crate` at the root of
the create's module structure.

```text
crate
    └── front_of_house
        ├── hosting
        │   ├── add_to_waitlist
        │   └── seat_at_table
        └── serving
            ├── take_order
            ├── serve_order
            └── take_payment
```

## Paths for Referring to an Item in the Module Tree

A path can take two forms:

- An absolute path starts from a crate root by using a crate name or a literal `crate`.
- A relative path starts from the current module and uses `self`, `super`, or an identifier in the current module.

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

The code above will not compile due to `hosting` and `fn add_to_waitlist()` not set as public (private).

### Exposing Paths with the `pub` Keyword

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

The code above will compile.

`front_of_house` is private, but because `eat_at_restaurant` function and `front_of_house` are siblings, `front_of_house` can be referred from `eat_at_restaurant`.

Each nested modules and functions need to be marked as public for external access.

### Starting Relative Paths with `super`

`super` is similar to `..` syntax in filesystem path.

```rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}
}
```

### Making Structs and Enums Public

If we use `pub` to designate structs, we still need to mark each field as public on a case-by-case basis.

If a struct has a private field, the struct needs to provide a public associated function that constructs an instance of struct. (`summer` function in example below)

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries");
}
```

If we make an enum public, all of its variants are public.

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

## Bring Paths Into Scope with `use` Keyword

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

// Use absolute path
use crate::front_of_house::hosting;
// Use relative path
// use self::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

### Idiomatic `use`

When bringing function into scope, it's idiomatic to bring the function's parent module into scope (like example above), so we have to specify the parent module when calling function,
making it clear that function isn't locally defined. The example below brings the function directly into scope, which is also a valid `use`.

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
}
```

When bringing in structs, enums and other items with `use`, it's idiomatic to specify the full path.

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

Exception to this idiom is if we're bringing two items with the same name into scope with `use` statements, because Rust does not allow it.

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
}

fn function2() -> io::Result<()> {
    // --snip--
}
```

### Providing New Names with `as` Keyword

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

### Re-exporting Names with `pub use`

When importing a name into scope with `use`, the name is private. We can re-export an item by bring it into scope but also making it available for others to bring into their scope.

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

### Using External Packages

Take `rand` package for example, to use `rand` in our project, we add this line to `Cargo.toml`:

```toml
rand = "0.8.3"
```

To bring `rand` definitions into scope of our package, we add a `use` line starting with the name of the crate, `rand`, and listed the items we wanted to bring into scope.
In the example below, the `Rng` trait is imported so we can use `thread_rng` function.

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..101);
}
```

Note that standard library (`std`) is also a create that's external to our package. Because it's shipped with Rust language, we don't need to add it to `Cargo.toml`,
but we need to refer to it with `use`.

```rust
use std::collections::HashMap;
```

### Using Nested Paths to Clean Up Large `use` Lists

```rust
use std::cmp::Ordering;
use std::io;
// is equivalent to
use std::{cmp::Ordering, io};
```

```rust
use std::io;
use std::io::Write;
// is equivalent to
use std::io::{self, Write};
```

### The Glob Operator

```rust
use std::collections::*;
```

## Separating Modules into Different Files

Original file (`src/lib.rs`)

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

Separate out `front_of_house` module into its own file.

`src/lib.rs`:

```rust
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

`src/front_of_house.rs`:

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

Using semicolon after `mod front_of_house` rather than a block tells Rust to load the contents of the module from another file with the same name.

We can separate modules further by extracting `hosting` module to its own file, and change `src/front_of_house.rs` to contain only the declaration of `hosting` module.

`src/front_of_house.rs`

```rust
pub mod hosting;
```

Extract `hosting` module to a new file.

`src/front_of_house/hosting.rs`

```rust
pub fn add_to_waitlist() {}
```

The module tree remains the same, and the function calls in `eat_at_restaurant` will work without any modification, even though the definitions live in different files.

This technique lets you move modules to new files as they grow in size.
