<!-- markdownlint-disable MD013 -->

# Enums and Pattern Matching

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Enums and Pattern Matching](#enums-and-pattern-matching)
  - [Enum Values](#enum-values)
    - [`Options` Enum](#options-enum)
  - [`match` Control Flow Operator](#match-control-flow-operator)
    - [Matching with `Option<T>`](#matching-with-optiont)
    - [Catch-all Patterns](#catch-all-patterns)
  - [Concise Control Flow with `if let`](#concise-control-flow-with-if-let)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Enum Values

```rust
enum IpAddrKind {
    V4,
    V6,
}
fn route(ip_kind: IpAddrKind) {...}
fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
    route(four);
    route(six);
}
```

Various ways to use enums:

```rust
enum IpAddr {
    V4(String),
    V6(String),
}
let home = IpAddr::V4(String::from("127.0.0.1"));
let loopback = IpAddr::V6(String::from("::1"));
```

Variants with different types:

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}
let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

Variants type defined as structs:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Defining an enum with variants is similar to defining different kinds of struct definitions.
This allows us to define to take multiple types of variants.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // method body would be defined here
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

### `Options` Enum

Rust does not have nulls, but it does have an enum that can encode concept of value being present or absent, which is defined by standard library:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`T` is a generic type parameter

```rust
let some_number = Some(5); // type Option<i32>
let some_string = Some("a string"); // type Option<&str>
let absent_number: Option<i32> = None; // We need to specify type since compiler cannot infer
```

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);
let sum = x + y; // Compile error: no implementation for `i8 + Option<i8>`
```

We need to convert `Option<T>` to `T` before we can perform `T` operations.

## `match` Control Flow Operator

```rust
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {}", state);
            25
        },
    }
}
```

### Matching with `Option<T>`

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}
let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

Matches are exhaustive, following code would not compile:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x { // Compile error: pattern `None` not covered
        Some(i) => Some(i + 1),
    }
}
```

### Catch-all Patterns

Catch all pattern: (`other` is chosen name)

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    other => move_player(other),
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}
```

`_`, a special pattern that matches any value and does not bind to that value.
This tells Rust we aren't going to use the value, so Rust won't warn us about an unused variable.

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => reroll(),
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn reroll() {}
```

No action can be expressed by using unit value

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => (),
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
```

## Concise Control Flow with `if let`

`if let` let you handle values that match one pattern while ignoring the rest.

```rust
let config_max = Some(3u8);
match config_max {
    Some(max) => println!("The maximum is configured to be {}", max),
    _ => (),
}
// can be shortened to
if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
}
```

we can use `else` with `if let`

```rust
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
// is equivalent to
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```
