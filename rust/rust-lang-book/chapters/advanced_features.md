<!-- markdownlint-disable MD013 -->

# Advanced Features

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Advanced Features](#advanced-features)
  - [Unsafe Rust](#unsafe-rust)
    - [Unsafe Superpowers](#unsafe-superpowers)
    - [dereferencing a Raw Pointer](#dereferencing-a-raw-pointer)
    - [Calling an Unsafe Function or Method](#calling-an-unsafe-function-or-method)
      - [Creating a Safe Abstraction over Unsafe Code](#creating-a-safe-abstraction-over-unsafe-code)
      - [Using `extern` Functions to Call External Code](#using-extern-functions-to-call-external-code)
        - [Calling Rust Functions from Other Languages](#calling-rust-functions-from-other-languages)
    - [Accessing or Modifying a Mutable Static Variable](#accessing-or-modifying-a-mutable-static-variable)
    - [Implementing an Unsafe Trait](#implementing-an-unsafe-trait)
    - [Accessing Fields of a Union](#accessing-fields-of-a-union)
  - [Advanced Traits](#advanced-traits)
    - [Specifying Placeholder Types in Trait Definitions with Associated Types](#specifying-placeholder-types-in-trait-definitions-with-associated-types)
    - [Default Generic Type Parameters and Operator Overloading](#default-generic-type-parameters-and-operator-overloading)
    - [Fully Qualified Syntax for Disambiguation](#fully-qualified-syntax-for-disambiguation)
    - [Using Supertraits to Require One Trait's Functionality Within Another Trait](#using-supertraits-to-require-one-traits-functionality-within-another-trait)
    - [Using the Newtype Pattern to Implement External Traits on External Types](#using-the-newtype-pattern-to-implement-external-traits-on-external-types)
  - [Advanced Types](#advanced-types)
    - [Using the Newtype Pattern for Type Safety and Abstraction](#using-the-newtype-pattern-for-type-safety-and-abstraction)
    - [Creating Type Synonyms with Type Aliases](#creating-type-synonyms-with-type-aliases)
    - [The Never Type that Never Returns](#the-never-type-that-never-returns)
    - [Dynamically Sized Types and the `Sized` Trait](#dynamically-sized-types-and-the-sized-trait)
  - [Advanced Functions and Closures](#advanced-functions-and-closures)
    - [Function Pointers](#function-pointers)
    - [Returning Closures](#returning-closures)
  - [Macros](#macros)
    - [The Difference Between Macros and Functions](#the-difference-between-macros-and-functions)
    - [Declarative Macros for General Metaprogramming](#declarative-macros-for-general-metaprogramming)
    - [Procedural Macros for Generating Code from Attributes](#procedural-macros-for-generating-code-from-attributes)
    - [How to Write a Custom `derive` Macro](#how-to-write-a-custom-derive-macro)
    - [Attribute-like Macros](#attribute-like-macros)
    - [Function-like Macros](#function-like-macros)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Unsafe Rust

Unsafe Rust exists because static analysis is conservative; it's better for compiler to reject some valid programs than to accept some invalid programs. However, unsafe Rust when done incorrectly, can cause problems due to memory unsafety, such as null pointer dereferencing.

### Unsafe Superpowers

To switch to unsafe Rust, use the `unsafe` keyword and then then start a new block that holds the unsafe code. There are 5 actions we can take in unsafe Rust:

- Dereference a raw pointer
- Call an unsafe function or method
- Access or modify a mutable static variable
- Implement an unsafe trait
- Access fields of `union`s

It's important to understand `unsafe` does not turn off the borrow checker or disable any other Rust's safety checks. The `unsafe` keyword only gives you access to above five features.

`unsafe` code does not mean it's dangerous, just that it's programmer's responsibility that the code access memory in a valid way. In addition, `unsafe` block makes it easier to identify errors with memory safety because they must be in `unsafe` block.

To isolate unsafe code as much as possible, it's best to enclose unsafe code within a safe abstraction and provide a safe API, which prevents uses of `unsafe` from leaking out. Part of the standard library are implemented as safe abstractions over unsafe code that has been audited.

### dereferencing a Raw Pointer

Unsafe Rust has two new types called _raw pointers_ that are similar to references. Raw pointers can be immutable or mutable and are written as `*const T` and `*mut T`. Similar to C/C++, the asterisk is not the dereference operator, it's part of the type name. In the context of raw pointers, _immutable_ means the pointer can't be directly assigned to after being dereferenced.

Different from references and smart pointers, raw pointers:

- Are allowed to ignore the borrowing rules by having both immutable and mutable pointers or multiple mutable pointers to the same location
- Are not guaranteed to point to valid memory
- Are allowed to be null
- Don't implement any automatic cleanup

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
```

We don't need to include `unsafe` keyword for above raw pointers, because raw pointer creation is safe, but we need to dereference raw pointers inside an unsafe block.

Because the above raw pointers are created from a valid reference, the raw pointers are valid. However, we cannot make such assumption about any pointers. Below is an example.

```rust
let address = 0x012345usize;
let r = address as *const i32;
```

Dereferencing raw pointers must be done in an `unsafe` block.

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

Note that the above code would not have compiled because `r1` and `r2` raw pointers are immutable and mutable reference to `num`, which offends Rust's ownership rules, but it is allowed in `unsafe` block.

Why use raw pointers? Once major use case is for interfacing with C code, which we will discuss later.

### Calling an Unsafe Function or Method

Unsafe functions and methods have an extra `unsafe` before the rest of definition. In addition, unsafe functions can only be called in an `unsafe` block.

```rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
```

If we try to call `dangerous` without the `unsafe` block, we'll get an error:

```text
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0133]: call to unsafe function is unsafe and requires unsafe function or block
 --> src/main.rs:4:5
  |
4 |     dangerous();
  |     ^^^^^^^^^^^ call to unsafe function
  |
  = note: consult the function's documentation for information on how to avoid undefined behavior

For more information about this error, try `rustc --explain E0133`.
error: could not compile `unsafe-example` due to previous error
```

#### Creating a Safe Abstraction over Unsafe Code

Wrapping unsafe code in a safe function is a common abstraction. As an example, the `split_at_mut` function from the standard library requires some unsafe code; it takes one mutable slice and splits it into two slice at the index given as an argument.

Usage:

```rust
let mut v = vec![1, 2, 3, 4, 5, 6];

let r = &mut v[..];

let (a, b) = r.split_at_mut(3);

assert_eq!(a, &mut [1, 2, 3]);
assert_eq!(b, &mut [4, 5, 6]);
```

It's not possible to implement `split_at_mut` using only safe Rust. The example implement the function for only slices of `i32` rather than generic type `T` for demonstration purposes.

```rust
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();

    assert!(mid <= len);

    (&mut values[..mid], &mut values[mid..])
}
```

Compilation fails:

```text
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0499]: cannot borrow `*values` as mutable more than once at a time
 --> src/main.rs:6:31
  |
1 | fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  |                         - let's call the lifetime of this reference `'1`
...
6 |     (&mut values[..mid], &mut values[mid..])
  |     --------------------------^^^^^^--------
  |     |     |                   |
  |     |     |                   second mutable borrow occurs here
  |     |     first mutable borrow occurs here
  |     returning this value requires that `*values` is borrowed for `'1`

For more information about this error, try `rustc --explain E0499`.
error: could not compile `unsafe-example` due to previous error
```

Rust's borrow checker can't understand that we are borrowing different parts of the slice; it only knows that we're borrowing from the same slice twice.

Below is an implementation using `unsafe` block, a raw pointer, and some calls to unsafe functions.

```rust
use std::slice;

fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

The `slice::from_raw_parts_mut` function takes a raw pointer and a length, and it creates a slice. The `add` method on `ptr` moves the raw pointer to mid.

Both `slice::from_raw_parts_mut` and `add` are unsafe because they must trust that the pointers are valid. Therefore, we had to put them in `unsafe` block.

Note that `split_at_mut` function does not need to be marked as `unsafe`, and we can call this function from safe Rust; essentially this is a safe abstraction to the unsafe code.

In contrast, here's an invalid use of `slice::from_raw_parts_mut`.

```rust
use std::slice;

let address = 0x01234usize;
let r = address as *mut i32;

let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
```

There is no guarantee that the memory referenced in above code contains valid `i32` values, so it would result in undefined behavior.

#### Using `extern` Functions to Call External Code

Rust has keyword `extern` that facilitates the creation and use of a _Foreign Function Interface (FFI)_.

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

##### Calling Rust Functions from Other Languages

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

The `[no_mangle]` annotation tells Rust compiler not to mangle the function name. _Mangling_ is when compiler changes the function names to different name that contains more information for other parts of the compilation process to consume but is less human readable. This usage of `extern` does not require `unsafe`.

### Accessing or Modifying a Mutable Static Variable

In Rust, global variables are called _static_ variables.

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

The names of static variables are in `SNAKE_CASE` by convention. Static variables can only store references with the `'static` lifetime, which is inferred by the Rust compiler.

A subtle difference between constants and immutable static variables is that values in a static variable have a fixed address in memory. Constants are allowed to duplicate their data whenever they're used. Another difference is that static variables can be mutable.

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

To define a mutable static variable, use the `mut` keyword. Any code that reads or writes from static variable msut be within an `unsafe` block because it's difficult to ensure there are no data races. When possible, it's preferable to use concurrency techniques and thread-safe smart pointers.

### Implementing an Unsafe Trait

We can use `unsafe` to implement an unsafe trait. A trait is unsafe when at least one of its methods has some invariant that the compiler can't verify.

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}

fn main() {}
```

By using `unsafe impl`, we promise that we'll uphold the invariants that the compiler can't verify.

As an example, recall `Sync` and `Send` marker trait discussed in concurrency chapter. The compiler automatically implements these traits if a type is composed entirely of `Send` and `Sync` types. If we implement a type that contains a type that is not `Send` or `Sync` such as raw pointers, and we want to mark that type as `Send` and `Sync`, we must use `unsafe` as Rust cannot verify that our type upholds the guarantees that it can be safely sent across threads or accessed from multiple threads.

### Accessing Fields of a Union

A `union` is similar to `struct`, but only one declared field is used in a particular instance at one time. Unions are primarily used to interface with unions in C code. Access union fields is unsafe because Rust can't guarantee the type of the data currently being stored in the union instance. Learn more about unions in the [Rust Reference](https://doc.rust-lang.org/reference/items/unions.html).

## Advanced Traits

### Specifying Placeholder Types in Trait Definitions with Associated Types

_Associated types_ are type place holders within a trait that can be used in trait method signatures. The implementor of a trait will specify the concrete type; this allows us to define a trait that uses some types without needing to know the exact types until the trait is implemented.

One example of a trait with an associated type is the `Iterator` trait that standard library provides.

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

Implementors of the `Iterator` trait will specify the concrete type for `Item`, and `next` method will return an `Option` containing a value of that concrete type.

Associated types is similar to generics, but there are differences.

Associated types:

```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
```

Generics:

```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

Difference is that when using generics, we must annotate the types in each implementation. This means when a trait has a generic parameter, it can be implemented for a type multiple times. When using `next` method on `Counter`, we would have to provide type annotations to indicate which implementation of `Iterator` we want to use.

With associated types, we don't need to annotate types because we can't implement a trait on a type multiple times. In the `Iterator` example, we can only choose what the type of `Item` will be once, because there can only be one `impl Iterator for Counter`.

### Default Generic Type Parameters and Operator Overloading

We can specify a default concrete type for generic type parameter. Syntax: `<PlaceholderType=ConcreteType>`.

An example of a situation where this technique is useful is with _operator overloading_, in which we customize the behavior of an operator. (such as `+`)

Rust does not allow creation of custom operators or overload arbitrary operators, but we can overload operations and corresponding traits listed in `std::ops` by implementing the traits associated with the operator.

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

The `Add` trait has an associated type named `Output` that determines the type returned from the `add` method.

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

The `Rhs=Self` is called _default type parameters_. The `Rhs` generic type parameter defines the type of the `rhs` parameter in `add` method. If no concrete type is specified for `Rhs` when implementing `Add` trait, the type of `Rhs` will default to `Self`, which is the type we implement `Add` on.

Below is an example of implementing the `Add` trait with customized `Rhs` type.

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

There are two ways to use default type parameters:

- To extend a type without breaking existing code
- To allow customization in specific cases most users won't need

### Fully Qualified Syntax for Disambiguation

Rust does not prevent a trait from having a method with the same name as another trait's method, or prevent us from implementing both traits on one type.

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```

To call the `fly` methods from `Pilot` trait, `Wizard` trait, or struct itself, we need to use more explicit syntax.

```rust
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

Program output:

```text
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.46s
     Running `target/debug/traits-example`
This is your captain speaking.
Up!
*waving arms furiously*
```

Because `fly` method takes a `self` parameter, if we had two types that both implement one trait, Rust can infer which implementation of a trait to use based on type of `self.

However, associated functions that are not methods don't have a `self` parameter. When there are multiple types or traits that define non-method functions with the same function name, Rust does not know which type to use unless we specify _fully qualified syntax_.

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
}
```

Program output:

```text
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.54s
     Running `target/debug/traits-example`
A baby dog is called a Spot
```

If we want to invoke `baby_name` method in `Animal` trait, we cannot directly invoke `Animal::baby_name()` as it does not have a `self` parameter, and there could be other types that implement the `Animal` trait.

```rust
fn main() {
    println!("A baby dog is called a {}", Animal::baby_name());
}
```

Compilation error:

```text
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0790]: cannot call associated function on trait without specifying the corresponding `impl` type
  --> src/main.rs:20:43
   |
2  |     fn baby_name() -> String;
   |     ------------------------- `Animal::baby_name` defined here
...
20 |     println!("A baby dog is called a {}", Animal::baby_name());
   |                                           ^^^^^^^^^^^^^^^^^ cannot call associated function of trait
   |
help: use the fully-qualified path to the only available implementation
   |
20 |     println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
   |                                           +++++++       +

For more information about this error, try `rustc --explain E0790`.
error: could not compile `traits-example` due to previous error
```

We can disambiguate the method call using fully qualified syntax.

```rust
fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

Program output:

```text
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/traits-example`
A baby dog is called a puppy
```

In general, fully qualified syntax is defined as follows:

```rust
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

For associated functions that are not methods, there would not be a `receiver`. we can use fully qualified syntax everywhere we call functions or methods, but we are allowed to omit any part of this syntax that Rust can infer.

### Using Supertraits to Require One Trait's Functionality Within Another Trait

If we write a trait definition that depends on another trait, we can require downstream type to implement the other trait. The trait our trait definition is relying on is called a _supertrait_ of our trait.

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

The above code requires type that implements `OutlinePrint` to also implement `Display` trait.

```rust
struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}
```

The above code would fail to compile as `Point` has not implemented `Display`.

```text
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0277]: `Point` doesn't implement `std::fmt::Display`
  --> src/main.rs:20:6
   |
20 | impl OutlinePrint for Point {}
   |      ^^^^^^^^^^^^ `Point` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Point`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
note: required by a bound in `OutlinePrint`
  --> src/main.rs:3:21
   |
3  | trait OutlinePrint: fmt::Display {
   |                     ^^^^^^^^^^^^ required by this bound in `OutlinePrint`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `traits-example` due to previous error
```

To fix compilation error, we implement `Display` on `Point`.

```rust
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

### Using the Newtype Pattern to Implement External Traits on External Types

In Chapter 10 we mentioned the orphan rule which states we are only allowed to implement a trait on a type if either the trait or the type are local to our crate. It's possible to get around this restriction using the _newtype pattern_, which involves creating a new type in a tuple struct. The tuple struct has one field and is a thin wrapper around the type we want to implement a trait for.

_Newtype_ is a term that originates from the Haskell programming language. There is no runtime performance penalty for using this pattern, and wrapper type is elided at compile time.

Below is an example to implement `Display` trait for `Vec<T>`.

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

The downside of using this technique is that `Wrapper` is a new type, so it does not have the methods of the value it's holding. We would have to implement all the methods of `Vec<T>` directly on `Wrapper` such that methods delegate to `self.0` to allow us to treat `Wrapper` exactly like a `Vec<T>`. If we wanted the new type to have every method the inner type has, implementing the `Deref` trait on the `Wrapper` to return the inner type would be a solution. We can also just implement the methods we want manually if don't want the `Wrapper` type to have all the methods of the inner type.

## Advanced Types

### Using the Newtype Pattern for Type Safety and Abstraction

The newtype pattern can be used to statically enforce that values are never confused and indicating the units of a value. For instance, recall that we used `Millimeters` and `Meters` structs wrapped `u32` values in a newtype.

We can also use the newtype pattern to abstract away some implementation details of a type: the new type can expose a public API that is different from the API of the private inner type.

Newtypes can also hide internal implementation. For example, we could provide a `People` type to wrap a `HashMap<i32, String>` that stores a person's ID associated with their name. Code using `People` would only interact with the public API we provide. The newtype pattern is a lightweight way to achieve encapsulation to hide implementation details.

### Creating Type Synonyms with Type Aliases

Rust provides the ability to declare a _type alias_ to give an existing type another name. For this we use the `type` keyword.

```rust
type Kilometers = i32;
```

Note that `Kilometers` is not a separate new type. `Kilometers` values are treated the same as `i32` values.

```rust
type Kilometers = i32;

let x: i32 = 5;
let y: Kilometers = 5;

println!("x + y = {}", x + y);
```

Unlike newtype, type alias does not do type checking, so if we mix up `Kilometers` and `i32` values somewhere, the compiler will not give us an error.

The main use case for type alias is to reduce repetition. For instance, we might have a lengthy type: `Box<dyn Fn() + Send + 'static>`:

```rust
    let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));

    fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {
        // --snip--
    }

    fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {
        // --snip--
    }
```

Type alias makes this code more manageable by reducing the repetition.

```rust
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
    // --snip--
}

fn returns_long_type() -> Thunk {
    // --snip--
}
```

Type aliases are also commonly used with the `Result<T, E>` type for reducing repetition. Consider the `std::io` module in the standard library; I/O operations often return a `Result<T, E>` to handle situations when operations fail to work, and the module has a `std::io::Error` struct that represents all possible I/O errors.

```rust
use std::fmt;
use std::io::Error;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
```

The `Result<..., Error>` is repeated a lot. As such, `std::io` has this type alias declaration:

```rust
type Result<T> = std::result::Result<T, std::io::Error>;
```

The `Write` trait function signatures end up looking like this:

```rust
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}
```

### The Never Type that Never Returns

Rust has a special type named `!` that's known in type theory lingo as the _empty type_ because it has no value. In Rust world we call it the `never type` because it stands int he place of the return type when a function will never return.

```rust
fn bar() -> ! {
    // --snip--
}
```

The above code read as "the function `bar` returns never." Functions that return never are called _diverging functions_.

What use is a type we never create values for?

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

Recall that `match` arms must all return the same type. `continue` has a `!` value. When Rust computes the type of `guess`, it looks at both match arms: former with a value of `u32` while latter with a `!` value. Because `!` can never have a value, Rust decides that type of `guess` is `u32`.

The formal way of describing this behavior is that expressions of type `!` can be coerced into any other type. We are allowed to end this `match` arm with `continue` because `continue` does not return a value; instead, it moves control back to the top of the loop, so in the `Err` case, we never assign a value to `guess`.

The never type is useful with the `panic!` macro as well.

```rust
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
```

Rust sees that `val` has the type `T` and `panic!` has the type `!`, so the result of the overall `match` expression is `T`. This code works because `panic!` does not produce a value; it ends the program.

One final expression that has the type `!` is a `loop`:

```rust
print!("forever ");

loop {
    print!("and ever ");
}
```

The loop never ends, so `!` is the value of the expression.

### Dynamically Sized Types and the `Sized` Trait

Rust needs to know how much space to allocate for a value of a particular type. This causes confusions on the ceoncept of _dynamically sized types_ or _DSTs_ or _unsized types_, as values of these types has size known only at runtime.

For instance, `str` is a DST.

```rust
let s1: str = "Hello there!";
let s2: str = "How's it going?";
```

Both values of a type must use the same amount of memory, but the `str` values above have different length. In this case, we make the types of `s1` and `s2` a `&str` rather than a `str`. Recall from Chapter 4 that slice data structure just stores the starting position and the length of the slice. So although a `&T` is a single value that stores the memory address of where `T` is located, a `&str` is _two_ values: the address and its length, so we always know the size of a `&str` no matter how long the string it refers to is.

In general this is the way in which Dynamically sized types are used in Rust: they have an extra bit of metadata that stores the size of the dynamic information. The golden rule of dynamically sized types is that we must always put values of dynamically sized types behind a pointer of some kind.

We can combine `str` with all kinds of pointers; for example, `Box<str>` or `Rc<str>`. In fact, every trait is a dynamically sized type we can refer to by using the name of the trait, and we can create trait objects by combining traits with pointers; for example, `&dyn Trait`, `Box<dyn Trait>` or `Rc<dyn Trait>`.

To work with DSTs, Rust provides the `Sized` trait to determine whether or not a type's size is known at compile time. The trait is automatically implemented for everything whose size is known at compile time. In addition, Rust implicitly adds a bound on `Sized` to every generic function.

```rust
fn generic<T>(t: T) {
    // --snip--
}
```

The above code is actually treated as such:

```rust
fn generic<T: Sized>(t: T) {
    // --snip--
}
```

By default, generic functions will work only on types that have a known size at compile time, but the restriction can be relaxed using special syntax.

```rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

A trait bound on `?Sized` means "`T` may or may not be `Sized`". The `?Trait` syntax is only available for `Sized` and not any other traits.

Also note that we switched the type of the `t` parameter from `T` to `&T`. Because the type might not be `Sized`, we need to use it behind some kind of pointer.

## Advanced Functions and Closures

### Function Pointers

Functions coerce to the type `fn` (not to be confused with the `Fn` closure trait). The `fn` type is called a _function pointer_.

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

Program output:

```text
The answer is: 12
```

Unlike closures, `fn` is a type rather than a trait, so we specify `fn` as the parameter type directly rather than declaring a generic type parameter with one of the `Fn` traits as a trait bound.

Function pointers implement all three of the closure traits (`Fn`, `FnMut`, `FnOnce`), meaning you can always pass a function pointer as an argument for a function that expects a closure. It's best to write functions using a generic type and one of the closure traits so the functions can accept either functions or closures.

One example of where we would want to only accept `fn` and not closures is when interfacing with external code that does not have closures: C functions can accept functions as arguments, but C does not have closures.

Below is an example of where we could use either a closure defined inline or a named function.

```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers.iter().map(|i| i.to_string()).collect();
```

```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers.iter().map(ToString::to_string).collect();
```

Recall that name of each enum variant defined also becomes an initializer function. We can use these initializer functions as function pointers that implement the closure traits.

```rust
enum Status {
    Value(u32),
    Stop,
}

let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
```

### Returning Closures

Closures are represented by traits, which means we can't return closures directly. In most cases where we might want to return a trait, we can instead use the concrete type that implements the trait as the return value of the function, and we can't do that with closures because they don't have a concrete type.

```rust
fn returns_closure() -> dyn Fn(i32) -> i32 {
    |x| x + 1
}
```

Compilation error:

```text
$ cargo build
   Compiling functions-example v0.1.0 (file:///projects/functions-example)
error[E0746]: return type cannot have an unboxed trait object
 --> src/lib.rs:1:25
  |
1 | fn returns_closure() -> dyn Fn(i32) -> i32 {
  |                         ^^^^^^^^^^^^^^^^^^ doesn't have a size known at compile-time
  |
  = note: for information on `impl Trait`, see <https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits>
help: use `impl Fn(i32) -> i32` as the return type, as all return paths are of type `[closure@src/lib.rs:2:5: 2:8]`, which implements `Fn(i32) -> i32`
  |
1 | fn returns_closure() -> impl Fn(i32) -> i32 {
  |                         ~~~~~~~~~~~~~~~~~~~

For more information about this error, try `rustc --explain E0746`.
error: could not compile `functions-example` due to previous error
```

The error references `Sized` trait again because Rust does not know how much space it will need to store the closure. We could use a trait object to fix this:

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

## Macros

The term _macro_ refers to a family of features in Rust: _declarative_ macros with `macro_rules!` and 3 kinds of _procedural_ macros:

- Custom `#[derive]` macros that specify code added with the `derive` attribute used on structs and enums
- Attribute-like macros that define custom attributes usable on any item
- Function-like macros that look like function calls but operate on the tokens specified as their argument

### The Difference Between Macros and Functions

Fundamentally, macros are a way of writing code that writes other code, a.k.a _metaprogramming_. For example, the `derive` attribute that generates an implementation of various traits for us, or the `println!` and `vec!` macros, they all _expand_ to produce more code than the code we've written manually.

Differences between macros and functions:

- Function signature must declare the number of type of parameters, while macros can take a variable number of parameters.
- Macros are expanded before the compiler interprets the meaning of the code, so a macro can implmeent a trait on a given type for instance, while a function can't because it gets called at runtime and a trait needs to be implemented at compile time.
- Macro definitions are more complex than function definitions.
- Macros must be defined and brought into scope _before_ they are called in a file, as opposed to functions which can be defined and called anywhere.

### Declarative Macros for General Metaprogramming

The most widely used form of macros in Rust is the _declarative marco_, also sometimes referred to as "macros by example", "`macro_rules!` macros", or just plain "macros". At the core, declarative macros is a bit similar to `match` expression.

To define a macro, we use the `macro_rules!` construct. As an example, we'll attempt implementing a slightly simplified definition of the `vec!` macro.

Macro usage:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Simplified macro definition:

```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

**Note** that the actual definition of the `vec!` macro in the standard library includes code to preallocate the correct amount of memory up front.

The `#[macro_export]` annotation indicates that the macro is made available if wrapped crate is brought into scope. Without this annotation, the macro can't be brought into scope.

Start macro definition with `macro_rules!` and then the name of macro without exclamation mark. The structure in the `vec!` body is similar to the structure of a `match` expression. We have one arm with pattern `( $( $x:expr ),* )`, followed by `=>` and the block of code associated with the pattern.

In the pattern, we use `$` to declare a variable. THe dollar sign makes it clear this is a macro variable as opposed to a regular Rust variable. The next set of parentheses captures values that match the pattern within the parentheses for use in the replacement code. Within `$()` is `$x:expr` which matches any Rust expression and gives the expression the name `$x`. The comma following `$()` indicates that a literal comma separator character could optionally appear after the code that matches the code in `$()`. The `*` specifies that pattern matches zero or more of whatever precedes the `*`.

When calling this macro with `vec![1, 2, 3];`, the `$x` pattern matches three time with the three expressions `1`, `2`, and `3`. In the code body, `temp_vec.push()` within `$()*` is generated for each part that matches `$()` in the pattern. The `$x` is replaced with each expression matched. So the code generated that replaces this macro call will be the following:

```rust
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

To learn more about how to write macros, there are online documentations and resources, such as [The Little Book of Rust Macros](https://veykril.github.io/tlborm/).

### Procedural Macros for Generating Code from Attributes

The second form of macros is the _procedural macro_, which acts more like a function. It accept some code as an input, operate on that code, and produce some code as an output. There are 3 kinds of procedural macros which all work in a similar fashion:

- Custom derive
- Attribute-like
- Function-like

When creating procedural macros, the definitions must reside in their own create with a special crate type. This is for complex technical reasons that Rust team hopes to eliminate in the future.

```rust
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

The function that defines a procedural macro takes a `TokenStream` as an input and produces a `TokenStream` as an output. The `TokenStream` type is defined by the `proc_macro` create that is included with Rust and represents a sequence of tokens. The function also has attribute attached that specifies the type of procedural macro being created. We can have multiple types of procedural macros in the same crate.

### How to Write a Custom `derive` Macro

For this section, we'll create a crate named `hello_macro` that defines a trait named `HelloMacro` with one associated function named `hello_macro`. Rather than making users implement the `HelloMacro` trait for each of their types, we'll provide a procedural macro so users can annotate their type with `#[derive(HelloMacro)]` to get a default implementation of the `hello_macro` function. The default implementation will print `Hello, Macro! My name is TypeName!` where `TypeName` is the name of the type on which the trait has been defined.

Macro usage:

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
```

The code should output:

```text
Hello, Macro! My name is Pancakes!
```

The first step is to create a new library crate:

```sh
cargo new hello_macro --lib
```

In `src/lib.rs`:

```rust
pub trait HelloMacro {
    fn hello_macro();
}
```

We can't yet provide the `hello_macro` function with default implementation that will print the name of the type the trait is implemented on; Rust does not have reflection capabilities, so it cannot look up the type's name at runtime. We need a macro to generate code at compile time.

To define the procedural macro, we'll need to create a new crate. (This restriction may be lifted in the future). Convention for macro crates naming: for a crate named `foo`, a custom derive procedural macro crate is called `foo_derive`. We'll start a new crate called `hello_macro_derive` inside `hello_crate` project

```sh
cargo new hello_macro_derive --lib
```

The two crates are tightly related, so the macro crate is created within the directory of `hello_macro` crate. If we change trait definition of `hello_macro`, we'll have to change the implmeentation of the procedural macro as well. The two crates will need to be published or imported separately. We could have `hello_macro` crate use `hello_macro_derive` as a dependency and re-export the procedural macro code, but this makes it impossible for users to use `hello_macro` without the `derive` functionality imported.

We would need to declare the macro crate as a procedural macro crate, and we would also need functionality from `syn` and `quote` crates. Below is the _Cargo.toml_ file.

```toml
[lib]
proc-macro = true

[dependencies]
syn = "1.0"
quote = "1.0"
```

Place macro definition in _src/lib.rs_ in the macro crate.

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation
    impl_hello_macro(&ast)
}
```

This code won't compile until we implement `impl_hello_macro`. We have split the code into `hello_macro_derive` outer function which is responsible for parsing the `TokenStream`, and the `impl_hello_macro` inner function which is responsible for transforming the syntax tree. This structure of the outer function is the same for almost every procedural macro crate. The inner function will be different depending on the macro's purpose.

The `proc_macro` crate comes with Rust so we don't need to add it to dependencies in _Cargo.toml_. The `proc_marco` crate is the compiler's API that allows us to read and manipulate Rust code from our code.

The `syn` crate parses Rust code from a string into a data structure that we can perform operations on. The `quote` crate turns `syn` data structures back into Rust code.

The `hello_macro_derive` function will be called when a user of our library specifies `#[derive(HelloMacro)]` on a type, thanks to the `proc_macro_derive` annotation specified with name `HelloMacro`. The name does not need to match the trait name but it's a convention.

The `parse` function in `syn` takes a `TokenStream` and returns a `DeriveInput` struct representing the parsed Rust code. Below is an example when parsing `struct Pancakes;`.

```rust
DeriveInput {
    // --snip--
    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

The fields of this struct show that the Rust code parsed is a unit struct with the `ident` (identifier) of `Pancakes` struct. There are more fields on this struct; check the `syn` documentation for `DeriveInput` for more info.

Note that we are calling `unwrap` to cause `hello_macro_derive` function to panic if the call to the `syn::parse` function fails here. It's necessary for procedural macro to panic on errors because `proc_macro_derive` functions must return `TokenStream` rather than `Result` to conform to the procedural macro API. The example is simplified by using `unwrap`; in production code, we should provide more specific error messages by using `panic!` or `expect.

```rust
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
```

The `ast.ident` returns the `Ident` struct instance containing the name of the annotated type. In the macro usage example, the `ident` we get will have `ident` field with value of `"Pancakes"`. Thus, the `name` variable in `impl_hello_macro` will contain an `Ident` struct instance that, when printed, will be the string `"Pancakes"`.

The `quote!` macro let us define the Rust code we want to return. The compiler expects something different to the direct result of the `quote!` macro's execution, so we need to convert it to a `TokenStream` by calling the `into` method, which consumes the intermediate representation and returns the value of required `TokenStream` type.

The `quote!` macro also provides some very cool templating mechanics: we can enter `#name` and `quote!` will replace it with the value in the variable `name`. Check out the `quote` create's docs for a thorough introduction.

The `stringify!` macro used here is built into Rust. It takes a Rust expression and turns the expression into a string literal. For instance, `1 + 2` turns into `"1 + 2"`. This is different than `format!` or `println!`, which evaluate the expression and then turns the result into a `String`. There is a possibility that the `#name` input might be an expression to print literally, so we use `stringify!`. Using `stringify!` also saves an allocatino by converting `#name` to a string literal at compile time.

At this point, `cargo build` should complete successfully in both crates. To use the trait and macro crates, we can create a new binary project by running `cargo new pancakes`, and then add both crates as dependencies in the `pancakes` crate's _Cargo.toml_. If both crates are published on crates.io, they would be regular dependencies; otherwise, we can specify them as `path` dependencies as follows:

```toml
hello_macro = { path = "../hello_macro" }
hello_macro_derive = { path = "../hello_macro/hello_macro_derive" }
```

Put the macro usage example in _src/main.rs_ and run `cargo run`. It should print `Hello, Macro! My name is Pancakes!`. The implementation of `HelloMacro` trait from the procedural macro was included without the `pancakes` crate needing to implmeent it; the `#[derive(HelloMacro)]` added the trait implementation.

### Attribute-like Macros

Attribute-like macros are similar to custom derive macros, but instead of generating code for the `derive` attribute, they allow us to create new attributes. They are also more flexible: `derive` only works for structs and enums; attributes can be applied to other items as well, such as functions.

As an example, say we have an attribute named `route` that annotates functions when using a web application framework:

```rust
#[route(GET, "/")]
fn index() {
    // --snip--
}
```

The `#[route]` attribute would be defined by the framework as a procedural macro. The signature of the macro definition is shown below:

```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
    // --snip--
}
```

Here, we have two parameters of type `TokenStream`. The first is for the contents of the attribute: the `GET, "/"` part. The second is the body of the itme the attribute is attached to, which in this case, `fn index() {}` and the rest of the function's body.

### Function-like Macros

Function-like macros define macros that look like function calls. Similarly to `macro_rules!` macros, they are more flexible than functions. However `macro_rules!` macros can be defined only using the match-like syntax we discussed earlier. Function-like macros take a `TokenStream` parameter and their definition manipulates that `TokenStream` using Rust code as other two types of procedural macros do. An example usage of function-like marco:

```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

The `sql!` macro would be defined like this:

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```
