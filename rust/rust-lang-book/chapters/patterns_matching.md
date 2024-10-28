<!-- markdownlint-disable MD013 -->

# Patterns and Matching

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Patterns and Matching](#patterns-and-matching)
  - [All the Places Patterns Can Be Used](#all-the-places-patterns-can-be-used)
    - [`match` Arms](#match-arms)
    - [Conditional `if let` Expressions](#conditional-if-let-expressions)
    - [`while let` Conditional Loops](#while-let-conditional-loops)
    - [`for` Loops](#for-loops)
    - [`let` Statements](#let-statements)
    - [Function Parameters](#function-parameters)
  - [Refutability: Whether a Pattern Might Fail to Match](#refutability-whether-a-pattern-might-fail-to-match)
  - [Pattern Syntax](#pattern-syntax)
    - [Matching Literals](#matching-literals)
    - [Matching Named Variables](#matching-named-variables)
    - [Multiple Patterns](#multiple-patterns)
    - [Matching Ranges of Values](#matching-ranges-of-values)
    - [Destructuring to Break Apart Values](#destructuring-to-break-apart-values)
      - [Destructuring Structs](#destructuring-structs)
      - [Destructuring Enums](#destructuring-enums)
      - [Destructuring Nested Structs and Enums](#destructuring-nested-structs-and-enums)
      - [Destructuring Structs and Tuples](#destructuring-structs-and-tuples)
    - [Ignoring Values in a Pattern](#ignoring-values-in-a-pattern)
      - [Ignoring Entire Value](#ignoring-entire-value)
      - [Ignoring Parts of a Value](#ignoring-parts-of-a-value)
      - [Ignoring an Unused Variable](#ignoring-an-unused-variable)
      - [Ignoring Remaining Parts of a Value](#ignoring-remaining-parts-of-a-value)
    - [Extra Conditionals with Match Guard](#extra-conditionals-with-match-guard)
    - [_at_ binding](#at-binding)
<!--toc:end-->

<!-- prettier-ignore-end -->

## All the Places Patterns Can Be Used

### `match` Arms

```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

Example:

```rust
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Pattern `_` will match anything, but it never binds to a variable, so it's often used in the last match arm, or if we want to ignore any value not specified.

### Conditional `if let` Expressions

The `if let` expreession is used as a shorter way to write the equivalent of a `match` that only matches one case. It's also possible to mix and match `if let`, `else if`, and `else if let` expressions. This gives us more flexibility than `match`, and also Rust does not require that conditions in `if let`, `else if`, `else if let` arms relate to each other.

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

Program output:

```text
Using purple as the background color
```

The line `if let Ok(age) = age` introduces a new shadowed `age` variable that contains the value inside the `Ok` variant. This means we need to place the `if age > 30` condition within the block, because `age` can't be compared to 30 until it's shadowed, which does not happen until new scope starts.

Downside of `if let` expression is that compiler does not check for exhaustiveness, whereas `match` expression does.

### `while let` Conditional Loops

```rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

### `for` Loops

```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

Program output:

```text
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/patterns`
a is at index 0
b is at index 1
c is at index 2
```

### `let` Statements

A `let` statement also uses patterns.

```rust
let PATTERN = EXPRESSION;
```

This is evident when we use `let` to desctructure a tuple:

```rust
let (x, y, z) = (1, 2, 3);
```

If the number of elements in the pattern does not match the number of elements in tuple, the overall type won't match and compilation fails.

```rust
let (x, y) = (1, 2, 3);
```

Compilation error:

```text
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0308]: mismatched types
 --> src/main.rs:2:9
  |
2 |     let (x, y) = (1, 2, 3);
  |         ^^^^^^   --------- this expression has type `({integer}, {integer}, {integer})`
  |         |
  |         expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected tuple `({integer}, {integer}, {integer})`
             found tuple `(_, _)`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `patterns` due to previous error
```

To fix the error, we could ignore one or more values in the tuple using `_` or `..`, which will be discussed later in the chapter.

### Function Parameters

The function parameters are patterns as well.

```rust
fn foo(x: i32) {
    // code goes here
}
```

The `x` is the pattern, since we can use it to destructure tuple.

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

## Refutability: Whether a Pattern Might Fail to Match

Patterns come in two forms: refutable and irrefutable. Patterns that will match for any possible value passed are _irrefutable_. For instance, in the statement `let x = 5`, `x` is irrefutable because `x` matches anything and therefore cannot fail to match. Patterns that can fail to match for some possible value are _refutable_. An example would be expression `if let Some(x) = val` because if `val` is `None`, `Some(x)` pattern will not match.

Function parameters, `let` statements, and `for` loops can only accept irrefutable patterns. The `if let` and `while let` expressions accept refutable and irrefutable patterns, but the compiler warns against irrefutable patterns because by definition those expressions are intended to handle possible failure.

It's important to understand the distinction to respond to error message. For instance, assume we have following statement.

```rust
let Some(x) = some_option_value;
```

if `some_option_value` is `None`, it would fail to match the pattern `Some(x)`, which means the pattern is refutable, but `let` can only accept an irrefutable pattern, so compilation fails:

```text
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0005]: refutable pattern in local binding: `None` not covered
 --> src/main.rs:3:9
  |
3 |     let Some(x) = some_option_value;
  |         ^^^^^^^ pattern `None` not covered
  |
  = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant
  = note: for more information, visit https://doc.rust-lang.org/book/ch18-02-refutability.html
note: `Option<i32>` defined here
 --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:518:1
  |
  = note:
/rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:522:5: not covered
  = note: the matched value is of type `Option<i32>`
help: you might want to use `if let` to ignore the variant that isn't matched
  |
3 |     let x = if let Some(x) = some_option_value { x } else { todo!() };
  |     ++++++++++                                 ++++++++++++++++++++++
help: alternatively, you might want to use let else to handle the variant that isn't matched
  |
3 |     let Some(x) = some_option_value else { todo!() };
  |                                     ++++++++++++++++

For more information about this error, try `rustc --explain E0005`.
error: could not compile `patterns` due to previous error
```

We have a refutable pattern where an irrefutable pattern is needed. To fix the compilation failure, we can change the code that uses the pattern; instead of using `let`, we can use `if let`.

```rust
if let Some(x) = some_option_value {
    println!("{}", x);
}
```

If we give `if let` a pattern that always match, compiler will give a warning.

```rust
if let x = 5 {
    println!("{}", x);
};
```

Program output:

```text
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
warning: irrefutable `if let` pattern
 --> src/main.rs:2:8
  |
2 |     if let x = 5 {
  |        ^^^^^^^^^
  |
  = note: this pattern will always match, so the `if let` is useless
  = help: consider replacing the `if let` with a `let`
  = note: `#[warn(irrefutable_let_patterns)]` on by default

warning: `patterns` (bin "patterns") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.39s
     Running `target/debug/patterns`
5
```

For this reason, match arms must use refutable patterns, except for the last arm, which should match any remaining values with an irrefutable pattern.

## Pattern Syntax

### Matching Literals

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

### Matching Named Variables

Named variables are irrefutable patterns that match any value, but there is a complication when you use named variables in `match` expressions. Because `match` starts a new scope, variables declared as part of a pattern inside the `match` expression will shadow those with the same name outside the `match` construct.

```rust
let x = Some(5);
let y = 10;

match x {
    Some(50) => println!("Got 50"),
    Some(y) => println!("Matched, y = {y}"),
    _ => println!("Default case, x = {:?}", x),
}

println!("at the end: x = {:?}, y = {y}", x);
```

When the pattern in second match arm introduces a new variable named `y` that will match any value inside a `Some` value, because we are in a new scope inside the `match` expression, this is a new `y` variable, not the `y` declared outside of `match` expression. The new `y` binds to the inner value of `Some` in `x`, so the program output is `5`.

If `x` had been `None`, the pattern in last arm is matched. Because we did not introduce `x` variable in the pattern of arm, so `x` is still the outer `x`. In this case, program would print `Default case, x = None`.

When `match` expression is done, its scope ends, and so does the shadowed `y`. The last `println!` would produce `at the end: x = Some(5), y = 10`.

To create a `match` expression that compares the values of the outer `x` and `y`, rather than introducing a shadowed variable, we would need to use a match guard conditional instead, which we'll discuss later.

### Multiple Patterns

In `match` expressions, you can match multiple patterns using the `|` syntax, which is the pattern _or_ operator.

```rust
    let x = 1;

    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
```

### Matching Ranges of Values

The `..=` syntax allows us to match an inclusive range of values.

```rust
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}
```

The compiler checks that range is not empty at compile time, which is only possible on `char` or numeric values, so ranges are only allowed with numeric or `char` values.

```rust
let x = 'c';

match x {
    'a'..='j' => println!("early ASCII letter"),
    'k'..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```

### Destructuring to Break Apart Values

#### Destructuring Structs

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

Because of common usage `let Point { x: x, y: y } = p;` contains a lot of duplication, Rust also has shorthand this specific usage.

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

We can also destructure with litreal values as part of the struct pattern.

```rust
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {x}"),
        Point { x: 0, y } => println!("On the y axis at {y}"),
        Point { x, y } => {
            println!("On neither axis: ({x}, {y})");
        }
    }
}
```

Program output:

```text
On the y axis at 7
```

#### Destructuring Enums

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.");
        }
        Message::Move { x, y } => {
            println!("Move in the x direction {x} and in the y direction {y}");
        }
        Message::Write(text) => {
            println!("Text message: {text}");
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change the color to red {r}, green {g}, and blue {b}",)
        }
    }
}
```

Program output:

```text
Change the color to red 0, green 160, and blue 255
```

For enum variants without data, we can only match on the literal variant value (`Message::Quit`).

For struct-like enum variants, we can use struct destructuring pattern.

For tuple-like enum variants, we can use tuple destructuring pattern.

#### Destructuring Nested Structs and Enums

```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("Change color to hue {h}, saturation {s}, value {v}")
        }
        _ => (),
    }
}
```

#### Destructuring Structs and Tuples

It's possible to mix, match, and nest destructuring patterns in more complex ways.

```rust
let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
```

### Ignoring Values in a Pattern

There are a few ways to ignore entire values or parts of values in a pattern:

- Using the `_` pattern
- Using the `_` pattern within another pattern
- Using a name that starts with an underscore
- Using `..` to ignore remaining parts of a value.

#### Ignoring Entire Value

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

#### Ignoring Parts of a Value

```rust
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}

println!("setting is {:?}", setting_value);
```

We can also use underscores in multiple places within one pattern.

```rust
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {first}, {third}, {fifth}")
        }
    }
```

#### Ignoring an Unused Variable

If we create a variable but don't use it anywhere, Rust will issue a warning because an unused variable could be a bug. However, sometimes it's useful for prototyping or other use cases. In those situations, we can tell Rust to not give us warning by starting name of the variables with `_`.

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

Note that there is subtle difference between using only `_` and using a name that starts with `_`. Syntax `_x` still binds the value to the variable, whereas `_` does not bind at all.

```rust
let s = Some(String::from("Hello!"));

if let Some(_s) = s {
    println!("found a string");
}

println!("{:?}", s);
```

The code above will fail compilation because `s` value is moved to `_s`, so it cannot be used again in outer scope `println!`. On the other hand, matching with `_` compiles, because `s` does not get moved into `_`.

```rust
let s = Some(String::from("Hello!"));

if let Some(_) = s {
    println!("found a string");
}

println!("{:?}", s);
```

#### Ignoring Remaining Parts of a Value

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

The `..` syntax will expand to as many values as it needs to be. Below is an example with tuple.

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {first}, {last}");
        }
    }
}
```

Using `..` must be unambiguous, otherwise the code will not compile. Below is an example.

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {
            println!("Some numbers: {}", second)
        },
    }
}
```

```text
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error: `..` can only be used once per tuple pattern
 --> src/main.rs:5:22
  |
5 |         (.., second, ..) => {
  |          --          ^^ can only be used once per tuple pattern
  |          |
  |          previously used here

error: could not compile `patterns` due to previous error
```

### Extra Conditionals with Match Guard

A _match guard_ is an additional `if` condition specified after a `match` arm pattern.

```rust
let num = Some(4);

match num {
    Some(x) if x % 2 == 0 => println!("The number {} is even", x),
    Some(x) => println!("The number {} is odd", x),
    None => (),
}
```

In previous sections we mentioned match guards can be used to solve pattern-shadowing problem. Here is an example of using match guards to prevent pattern-shadowing.

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {n}"),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {y}", x);
}
```

It's also possible to use _or_ operator `|` in a match guard to specify multiple patterns.

```rust
let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"),
}
```

Note that the or operator `|` takes precedence over the match guard `if` condition, so the match arm behaves more like `(4 | 5 | 6) if y =>`

### _at_ binding

The _at_ operator `@` lets us create a variable that holds a value at the same time as we're testing that value for a pattern match.

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello {
        id: id_variable @ 3..=7,
    } => println!("Found an id in range: {}", id_variable),
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    }
    Message::Hello { id } => println!("Found some other id: {}", id),
}
```

By specifying `id_variable @` before the range `3..=7`, we captured whatever value matched the range while also testing that value matched the range pattern.

In second match arm, we are able to test value matched range pattern, but pattern code is not able to use the value from `id` field. In the third match arm, the value is available to use in the arm's code in a variable named `id`, but we have not applied any test to the value in the `id` field.
