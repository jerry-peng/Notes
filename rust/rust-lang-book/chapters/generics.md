<!-- markdownlint-disable MD013 -->
<!-- markdownlint-disable MD029 -->

# Generic Types, Traits, and Lifetimes

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Generic Types, Traits, and Lifetimes](#generic-types-traits-and-lifetimes)
  - [Generic Data Types](#generic-data-types)
    - [In Function Definitions](#in-function-definitions)
    - [In Struct Definitions](#in-struct-definitions)
    - [In Enum Definitions](#in-enum-definitions)
    - [In Method Definitions](#in-method-definitions)
    - [Performance of Code Using Generics](#performance-of-code-using-generics)
  - [Traits: Defining Shared Behavior](#traits-defining-shared-behavior)
    - [Defining a Trait](#defining-a-trait)
    - [Implementing a Trait on a Type](#implementing-a-trait-on-a-type)
    - [Default Implementations](#default-implementations)
    - [Traits as Parameters](#traits-as-parameters)
      - [Trait Bound Syntax](#trait-bound-syntax)
      - [Specifying Multiple Trait Bound](#specifying-multiple-trait-bound)
      - [Clearer Trait Bounds with `where` Clauses](#clearer-trait-bounds-with-where-clauses)
    - [Returning Types that Implement Traits](#returning-types-that-implement-traits)
    - [Fixing the `largest` Function with Trait Bounds](#fixing-the-largest-function-with-trait-bounds)
    - [Using Trait Bounds to Conditionally Implement Methods](#using-trait-bounds-to-conditionally-implement-methods)
  - [Validating References with Lifetimes](#validating-references-with-lifetimes)
    - [Preventing Dangling References With Lifetimes](#preventing-dangling-references-with-lifetimes)
    - [The Borrow Checker](#the-borrow-checker)
    - [Generic Lifetimes in Functions](#generic-lifetimes-in-functions)
    - [Lifetime Annotation Syntax](#lifetime-annotation-syntax)
    - [Lifetime Annotation in Function Signatures](#lifetime-annotation-in-function-signatures)
    - [Thinking in Terms of Lifetimes](#thinking-in-terms-of-lifetimes)
    - [Lifetime Annotations in Struct Definitions](#lifetime-annotations-in-struct-definitions)
    - [Lifetime Elision](#lifetime-elision)
    - [Lifetime Annotations in Method Definitions](#lifetime-annotations-in-method-definitions)
    - [The Static Lifetime](#the-static-lifetime)
    - [Generic Type Parameters, Trait Bounds, and Lifetimes Together](#generic-type-parameters-trait-bounds-and-lifetimes-together)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Generic Data Types

### In Function Definitions

Here we have a function that return the largest number:

```rust
fn largest(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

If we want to implement similar function for `char`, we'd need to duplicate the function for `char` type:

```rust
fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

To eliminate duplication, we can use generic type parameter in a single function.

```rust
// ... snip
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

But this code will encounter compile error:

```text
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- T
  |            |
  |            T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
error: could not compile `chapter10` due to previous error
```

`std::cmp::PartialOrd` is a trait, which will be discussed in next _Traits: Defining Shared Behavior_ section. For now, the error means body of `largest` won't work for all possible types of `T`.

### In Struct Definitions

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
    let wont_work = Point { x: 5, y: 4.0 }; // Compile error
}
```

`x` and `y` values must be of same type or there will be compile error.

To define a `Point` struct where `x` and `y` are both generics but could have different types, we can use multiple generic type parameters.

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

### In Enum Definitions

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### In Method Definitions

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

Here, we've defined a method named `x` on `Point<T>` that returns a reference to the data in field `x`.

By declaring `T` as generic type after `impl`, Rust can identify the type in the angle brackets in `Point` as a generic type rather than a concrete type.
We could use a different name for generic parameter since it is re-declared, but using the same name is conventional.

The other option is defining methods on the type with some constraint on generic type.

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

This means type `Point<f32>` will have a method named `distance_from_origin` while other instance of `Point<T>` where `T` is not of type `f32` will not have this method defined.

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

Generic type parameters in a struct definition aren't always the same as those you use in that struct's method signatures.

### Performance of Code Using Generics

Rust implements generics in such as a way that code does not run any slower using generic types than it would with concrete types. It is accomplished by performing monomoprphization,
which is the process of turning generic code into specific code by filling concrete types that are used when compiled.

## Traits: Defining Shared Behavior

### Defining a Trait

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

### Implementing a Trait on a Type

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

We can call trait methods the same way we call regular methods. Assume `Summary` and `Tweet` are in `aggregator` library:

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    // print: "1 new tweet: horse_ebooks: of course, as you probably already know, people"
    println!("1 new tweet: {}", tweet.summarize());
}
```

Note that we can implement a trait on a type only if at least one of the trait or the type is local to our create. For example, we can implement standard library traits like `Display` on a
custom type like `Tweet` as part of our `aggregator` create functionality, because type `Tweet` is local to our `aggregator` create. We can also implement `Summary` on `Vec<T>` in our `aggregator`
create, because the trait `Summary` is local to our `aggregator` create.

But we cannot implement external traits on external types. For example, we can't implement the `Display` trait on `Vec<T>`. This rules ensures that other people's code cannot break your code
and vice versa. Without this rule, two crates could implement same external trait for same external type, and Rust wouldn't know which implementation to use.

### Default Implementations

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

impl Summary for NewsArticle {}

fn main() {
    let article = NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("Iceburgh"),
        content: String::from(
            "The Pittsburgh Penguins once again are the best \
             hockey team in the NHL.",
        ),
    };

    println!("New article available! {}", article.summarize());
    // Prints "New article available! (Read more...)"
}
```

Creating a default implementation for `summarize` doesn't require us to change implementation for `Summary` and `Tweet`, because the syntax for overriding a default implementation
and implementing a trait method are the same.

Default implementations can call other methods in same trait, even if they don't have default implementations.

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
    // Print "1 new tweet: (Read more from @horse_ebooks...)"
}
```

When we implement the trait on `Summary` type, we only need to define `summarize_author`.

Note it's not possible to call the default implementation from an overriding implementation of the same method.

### Traits as Parameters

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

The function parameter accepts any type that implements the specified trait. In the body of `notify`, we can call any methods on `item`that com from the `Summary` trait.

#### Trait Bound Syntax

`impl Trait` syntax is syntax sugar for longer form:

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

The `impl Trait` syntax is convenient and makes for more concise code in simple cases. Trait bound syntax can express more complex cases.

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}
```

If we want to allow `item1` and `item2` to have different types, using `impl Trait` would be appropriate (as long as both types implement `Summary`).

To force both parameters to have the same type, it's only possible through trait bound syntax:

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {}
```

#### Specifying Multiple Trait Bound

```rust
pub fn notify(item: &(impl Summary + Display)) {}
pub fn notify<T: Summary + Display>(item: &T) {}
```

When two bounds are specified, the body of `notify` can call `summarize` and use `{}` to format `item`.

#### Clearer Trait Bounds with `where` Clauses

Sometimes function with multiple generic type parameters can have many trait bound information and clutter function signature.
Rust has alternative syntax for specifying trait bounds inside a `where` clause after function signature.

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {

// is equivalent to:
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### Returning Types that Implement Traits

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

The ability to return a type that is only specified by the trait it implements is useful in the context of closures and iterators which are discussed in
_Functional Language Features: iterators and closures_ chapter.
Closures and iterators create types that only compiler knows and very long to specify.

Note that a function can only return a single type that implements a trait. The code below won't work:

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
}
```

To return different types that implement trait objects, go to _Using Trait Objects That Allow for Values of Different Types_ section of _Object Oriented Programming Features of Rust_ chapter.

### Fixing the `largest` Function with Trait Bounds

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

To compare using `>`, the operator is defined as a default method on the standard library trait `std::cmp::PartialOrd`, so we need to specify trait bound.
Additionally, `list[0]` will cause error: `cannot move out of type [T], a non-copy slice`. The function is generic, so it's possible for `list` parameter to have types in it
that don't implement the `Copy` trait, which means we cannot move value out of `list[0]` into `largest`, which resulted in this error, so we need to specify `Copy` trait bound.

If we don't want to restrict the `largest` function to values that implement the `Copy` trait, we could specify `T` has trait bound `Clone` instead of `Copy`. Then we could
clone each value in the slice when we want `largest` function to have ownership. Using `clone` function means we're potentially making more heap allocations.

Another way we could implement `largest` function is to return a reference to a `T` value in the slice. If we change return type to `&T` instead of `T`, we would't need the `Clone` or `Copy`
trait bounds and we could avoid heap allocations.

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    &largest
}
```

### Using Trait Bounds to Conditionally Implement Methods

By using trait bounds with an `impl` block that uses generic type parameters, we can implement method conditionally for types that implement the specified traits.

The code below implements `cmp_display` method if its inner type `T` implements `PartialOrd` and `Display` type.

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

We can also conditionally implement a trait for any type that implements another trait. Implementation of a trait on any type that satisfies the trait bounds are called _blanket implementations_
and are extensively used in Rust standard library.

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

This allows us to call `to_string` method defined by `ToString` trait on any type that implements the `Display` trait. For example:

```rust
let s = 3.to_string();
```

Another kind of generic that we've been using is called _lifetimes_. Rather than ensuring type has the behavior we want, lifetime ensures that references are valid as long as we need them to be.

## Validating References with Lifetimes

Most of the time, lifetimes are implicit and inferred, just like most of the time, types are inferred. We must annotate lifetimes when the lifetimes of references would be related in
a few different ways. Rust requires us to annotate relationships using generic lifetime parameters to ensure actual references used at runtime will definitely be valid.

### Preventing Dangling References With Lifetimes

```rust
{
    let r;

    {
        let x = 5;
        r = &x;
        // ^^ borrowed value does not live long enough
    }

    println!("r: {}", r);
}
```

This code won't compile because value that `r` is referring to has gone out of scope before it is used.

```text
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
5 |         let x = 5;
  |             - binding `x` declared here
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 |
9 |     println!("r: {r}");
  |                  --- borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` (bin "chapter10") due to 1 previous error
```

### The Borrow Checker

Rust compiler has a _borrow checker_ that compares scopes to determine whether all borrows are valid.

```rust
{
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```

At compile time, Rust compares the size of two lifetimes and sees that `r` has a lifetime of `'a` but it refers to memory with a lifetime of `'b`, and because the subject of reference does not
live as long as the reference, program is rejected.

```rust
{
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```

Here, `x` has longer lifetime than `r`, so `r` can reference `x`.

### Generic Lifetimes in Functions

Let's write a function that returns the longer of two string slices:

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

If we try to implement `longest` function, it won't compile:

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

```text
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ^^^^    ^^^^^^^     ^^^^^^^     ^^^

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` due to previous error
```

The help text says return type need generic lifetime parameter because Rust can't tell whether the reference being returned refers to `x` or `y`.

### Lifetime Annotation Syntax

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

### Lifetime Annotation in Function Signatures

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

The function signature tells Rust that for some lifetime `'a`, the function takes two parameters, both are string slices that live at least as long as lifetime `'a`,
and the string slice returned from the function will live as long as lifetime `'a`.
This means lifetime of reference returned by `longest` function is the same as the smaller of the lifetime of the references passed in.

Specifying lifetime parameters in function signature does not change lifetimes of values passed in or returned, instead, the borrow checker rejects any values that don't adhere to these constraints.

When we pass concrete references to `longest`, the concrete lifetime that is substituted for `'a` is the part of overlap scope between `x` and `y`.
In other word, the generic lifetime `'a` will get the concrete lifetime that is equal to the smaller of the lifetimes of `x` and `y`. Because we've annotated the returned references with
the same lifetime parameter `'a`, the returned reference will also be valid for the length of the smaller of the lifetimes of `x` and `y`.

```rust
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

`string1` is valid until end of outer scope, while `string2` is valid until end of inner scope. `result` reference something that is valid until the end of inner scope, which is valid.

If we run this code:

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

We get this error:

```text
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `string2` does not live long enough
 --> src/main.rs:6:44
  |
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!("The longest string is {}", result);
  |                                          ------ borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
```

The error shows that for `result` to be valid for the `println` statement, `string2` would need to be valid until the end of the outer scope.

### Thinking in Terms of Lifetimes

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

In this example, we specified lifetime parameter `'a` for parameter `x` and the return type, but not `y`,
because lifetime of `y` does not have any relationship with the lifetime of `x` or the return value.

When returning reference from a function, the lifetime parameter for the return type needs to match the lifetime parameter for one of the parameters.

This attempted implementation of `largest` function would not compile:

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

Error message:

```text
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0515]: cannot return value referencing local variable `result`
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ------^^^^^^^^^
   |     |
   |     returns a value referencing data owned by the current function
   |     `result` is borrowed here

For more information about this error, try `rustc --explain E0515`.
error: could not compile `chapter10` due to previous error
```

In this case, the best fix would be to return owned data type rather than a reference so the calling function is responsible for cleaning up the value.

### Lifetime Annotations in Struct Definitions

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

This annotation means an instance of `ImportantExcerpt` can't outlive the reference it holds in its `part` field.

### Lifetime Elision

Recall a function from a past chapter:

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

The reason this function does not need lifetime annotations is historical: in early version of Rust, the code above would not compile because every reference needed an explicit lifetime.
So the function signature would look like this:

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

Rust team found that Rust programmers were entering the same lifetime annotations repeatedly in particular situations that are predictable and followed a few deterministic patterns.
The developers programmed these patterns into compiler's code so borrow checker could infer lifetimes in these situations.
In the future, it's possible that more deterministic patterns will emerge and be added to the compiler.

The patterns programmed into Rust's analysis of references are called the _lifetime elision rules_.
These are a set of particular cases that the compiler will consider, and for any codes that fit these cases, explicit lifetimes are not needed.

Lifetimes on functions or method parameters are called _input lifetimes_ and lifetimes on return values are called _output lifetimes_.

There are 3 lifetimes elision rules:

1. Each parameter that is a reference gets its own lifetime parameter.

```rust
fn foo<'a>(x: &'a i32)
fn foo<'a, 'b>(x: &'a i32, y: &'b i32)
// and so on
```

2. If there is exactly one lifetime parameter, that lifetime is assigned to all output lifetime parameter.

```rust
fn foo<'a>(x: &'a i32) -> &'a i32
```

3. If there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` because this is a method, the lifetime of `self` is assigned to all output lifetime parameters.
   This rule makes methods much nicer to read and write.

Examples of how lifetime elision rules infer lifetime parameters:

```rust
fn first_word(s: &str) -> &str {
// apply first rule
fn first_word<'a>(s: &'a str) -> &str {
// apply second rule
fn first_word<'a>(s: &'a str) -> &'a str {
// third rule does not apply since this is not a method.
```

// A function

```rust
fn longest(x: &str, y: &str) -> &str {
// apply first rule
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
// second and third rule does not apply, and output lifetime cannot be inferred, so there will be compilation errors, we need to specify lifetime parameters explicitly.
```

### Lifetime Annotations in Method Definitions

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

There are two input lifetimes, so Rust applies first lifetime elision rule and give both `&self` and `announcement` their own lifetimes.
Because one of the parameter is `&self`, the return type gets the lifetime of `&self`, and all lifetimes have been accounted for.

### The Static Lifetime

```rust
let s: &'static str = "I have a static lifetime.";
```

Static lifetime denotes that affected reference can live for entire duration of the program. All string literals have static lifetime, because the string text
is stored directly in program's binary, which is always available.

### Generic Type Parameters, Trait Bounds, and Lifetimes Together

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
