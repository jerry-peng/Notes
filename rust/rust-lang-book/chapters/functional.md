<!-- markdownlint-disable MD013 -->

# Functional Language Features: Iterators and Closures

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Functional Language Features: Iterators and Closures](#functional-language-features-iterators-and-closures)
  - [Closures](#closures)
    - [Closure Type Inference and Annotation](#closure-type-inference-and-annotation)
    - [Capturing References or Moving Ownership](#capturing-references-or-moving-ownership)
    - [Moving Captured Values Out of Closures and the Fn Traits](#moving-captured-values-out-of-closures-and-the-fn-traits)
  - [Processing a Series of Items with Iterators](#processing-a-series-of-items-with-iterators)
    - [The Iterator Trait and the next Method](#the-iterator-trait-and-the-next-method)
    - [Methods that Consume the Iterator](#methods-that-consume-the-iterator)
    - [Methods that Produce Other Iterators](#methods-that-produce-other-iterators)
    - [Using Closures that Capture Their Environment](#using-closures-that-capture-their-environment)
  - [Comparing Performance: Loops vs. Iterators](#comparing-performance-loops-vs-iterators)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Closures

Rust's closures are anonymous functions you can save in a variable or pass as arguments to other functions.
You can create the closure in one place and then call the closure to evaluate it in a different context.
Unlike functions, closures can capture values from the scope in which they’re defined.

### Closure Type Inference and Annotation

Closures are typically shorter and relevant in narrower context when compared to functions. With these limited context, the compiler can mostly infer the types of parameters and return type with some exceptions.

Annotations can be added if we want explicitness at the cost of being more verbose than necessary.

Example:

```rust
let expensive_closure = |num: u32| -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

The closures/function all have the same behavior.

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

For closure definitions, compiler will infer one concrete type for each parameters and return type. For example, code below will get rejected by compiler.

```rust
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

Error:

```text
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
error[E0308]: mismatched types
 --> src/main.rs:5:29
  |
5 |     let n = example_closure(5);
  |             --------------- ^- help: try using a conversion method: `.to_string()`
  |             |               |
  |             |               expected `String`, found integer
  |             arguments to this function are incorrect
  |
note: expected because the closure was earlier called with an argument of type `String`
 --> src/main.rs:4:29
  |
4 |     let s = example_closure(String::from("hello"));
  |             --------------- ^^^^^^^^^^^^^^^^^^^^^ expected because this argument is of type `String`
  |             |
  |             in this closure call
note: closure parameter defined here
 --> src/main.rs:2:28
  |
2 |     let example_closure = |x| x;
  |                            ^

For more information about this error, try `rustc --explain E0308`.
error: could not compile `closure-example` (bin "closure-example") due to 1 previous error
```

### Capturing References or Moving Ownership

Closures can capture values from their environment in three ways, same as functions:

- borrowing immutably
- borrowing mutably
- taking ownership

Below is an example of immutable borrow in closure:

```rust
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");
    let only_borrows = || println!("From closure: {list:?}");
    println!("Before calling closure: {list:?}");
    only_borrows();
    println!("After calling closure: {list:?}");
}
```

```text
$ cargo run
     Locking 1 package to latest compatible version
      Adding closure-example v0.1.0 (/Users/carolnichols/rust/book/tmp/listings/ch13-functional-features/listing-13-04)
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
Before calling closure: [1, 2, 3]
From closure: [1, 2, 3]
After calling closure: [1, 2, 3]
```

Below is an example of mutable borrow in closure:

```rust
fn main() {
    let mut list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");
    let mut borrows_mutably = || list.push(7);
    borrows_mutably();
    println!("After calling closure: {list:?}");
}
```

```text
$ cargo run
     Locking 1 package to latest compatible version
      Adding closure-example v0.1.0 (/Users/carolnichols/rust/book/tmp/listings/ch13-functional-features/listing-13-05)
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
After calling closure: [1, 2, 3, 7]
```

Use `move` keyword before parameter list if you want to force closure to take ownership of values it uses in the environment even though the closure does not strictly need ownership. This is mostly useful when passing a closure to a new thread to move the data so that it's owned by the new thread.

Below is an example:

```rust
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");

    thread::spawn(move || println!("From thread: {list:?}"))
        .join()
        .unwrap();
}
```

The `move` is needed in this case because the main thread might finish before the new thread. If main thread maintains ownership of `list` but ended before new thread did and dropped `list`, the immutable reference in new thread would be invalid.

### Moving Captured Values Out of Closures and the Fn Traits

A closure body can do any of the following:

- move a captured value out of the closure
- mutate the captured value
- neither move or mutate the captured value
- capture nothing from the environment

The way closure captures and handles values from environment affects which traits the closure implements, and traits are how functions and structs can specify what kind of closures they can use.

Closures automatically implement one, two, or all three of these `Fn` traits, in an additive fashion:

- `FnOnce`: applies to closures that can be called once.
  - All closures implement at least this trait, because all closures can be called.
  - A closure that moves captured values out of its body will only implement `FnOnce` and none of the other `Fn` traits, because it can only be called once.
- `FnMut` applies to closures that don't move captured values out of their body, but that might mutate the captured values.
  - These closures can be called more than once.
- `Fn` applies to closures that don't move captured values out of their body and that don’t mutate captured values, as well as closures that capture nothing from their environment.
  - These closures can be called more than once without mutating their environment, which is important in cases such as calling a closure multiple times concurrently.

Below is the definition of `unwrap_or_else` method on `Option<T>`:

```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

The parameter `f` is closure we provide when calling `unwrap_or_else`. The generic type `F` is `FnOnce() -> T`, which means `F` must be able to be called at most once, take no arguments, and return a generic `T`. Because all closures implement `FnOnce`, `unwrap_or_else` accepts all three kinds of closures.

**Note:** Functions can implement all three of the `Fn` traits too. if what we want to do does not require capturing a value from the environment, we can use the name of function in place of a closure. For example, on an `Option<Vec<T>>` value, we could call `unwrap_or_else(Vec::new)` to get a new empty vector if value is `None`.

For `FnMut`, there is a standard library method `sort_by_key` that uses `FnMut` instead of `FnOnce` for the trait bound. Below is an example of `sort_by_key` that order `Rectangle` instances by `width` attribute from low to high.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    list.sort_by_key(|r| r.width);
    println!("{list:#?}");
}
```

The reason `sort_by_key` is defined to take an `FnMut` closure is that it calls the closure multiple times: once for each item in the slice.

Below is an example where we try to use a closure with `FnOnce` trait that moves a value out of the environment. Compiler will reject the program.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut sort_operations = vec![];
    let value = String::from("closure called");

    list.sort_by_key(|r| {
        sort_operations.push(value);
        r.width
    });
    println!("{list:#?}");
}
```

The closure implements `FnOnce` because it moves ownership of `value` to the `sort_operations` vector.

The `Fn` traits are important when defining or using functions or types that make use of closures.

## Processing a Series of Items with Iterators

In Rust, iterators are _lazy_, meaning they don't have effect until you call methods that consume the iterator.

In Chapter 3, we iterated over an array using a `for` loop to execute some code on each item. Under the hood this implicitly created and then consumed an iterator.

In languages without iterators, we would likely iterate through an array by starting at index 0, and so on. Iterators handle all of those logic for us, cutting down repetitive code.

### The Iterator Trait and the next Method

All iterators implement a trait named `Iterator` that is defined in the standard library. The definition of the trait looks like this:

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // methods with default implementations elided
}
```

Some new syntax: `type Item` and `Self::Item`, which are defining an _associated type_ with this trait, which will be discussed in Chapter 19. For now, all we need to know is this code says implementing the `Iterator` trait requires `Item` type to be defined, and the type is used as return type in `next` method.

Below is an example of what `next` method returns.

```rust
#[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
```

Note that we need to make `v1_iter` mutable: calling `next` method on an iterator changes internal state that the iterator uses to keep track of where it is in the sequence. If we use `for` loop to iterate through iterator, the iterator does not need to be mutable because for loop took ownership and made it mutable behind the scene.

`Item` ownership:

- The `iter` method produces an iterator over immutable references.
- The `iter_mut` method produces an iterator over mutable references.
- The `into_iter` method produces an iterator that takes ownership of item and returns value.

### Methods that Consume the Iterator

The `Iterator` trait has a number of different methods with default implementations provided by standard library, some of which call the `next` method, which is why `next` method is required to be implemented.

Methods that call `next` are called _consuming adaptors_, because calling them uses up the iterator. An example is the `sum` method, which takes ownership of iterator.

```rust
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];
        let v1_iter = v1.iter();
        let total: i32 = v1_iter.sum();
        assert_eq!(total, 6);
    }
```

We are not allowed to use `v1_iter` after call to `sum` because `sum` takes ownership of the iterator we call it on.

### Methods that Produce Other Iterators

_Iterator adaptors_ are methods defined on the Iterator trait that don’t consume the iterator. Instead, they produce different iterators by changing some aspect of the original iterator.

Below is an example using iterator adaptor method `map` and returns a new iterator that increment each item by 1.

```rust
    let v1: Vec<i32> = vec![1, 2, 3];
    v1.iter().map(|x| x + 1);
```

However, compiler returns a warning:

```text
$ cargo run
   Compiling iterators v0.1.0 (file:///projects/iterators)
warning: unused `Map` that must be used
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: iterators are lazy and do nothing unless consumed
  = note: `#[warn(unused_must_use)]` on by default
help: use `let _ = ...` to ignore the resulting value
  |
4 |     let _ = v1.iter().map(|x| x + 1);
  |     +++++++

warning: `iterators` (bin "iterators") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.47s
     Running `target/debug/iterators`
```

The iterator adaptors are lazy, and they need to be consumed to take effect. To fix the error, we can use `collect` method which consumes the iterator and collects the resulting values into a collection data type.

```rust
    let v1: Vec<i32> = vec![1, 2, 3];
    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
    assert_eq!(v2, vec![2, 3, 4]);
```

The iterator adaptors calls can be chained to perform complex actions in a readable way, but because all iterators are lazy, we would have to call a consuming adaptor method to get results from calls to iterator adaptors.

### Using Closures that Capture Their Environment

Below is an example that filter `Shoe` instances with specific `size` attribute.

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe {
                size: 10,
                style: String::from("sneaker"),
            },
            Shoe {
                size: 13,
                style: String::from("sandal"),
            },
            Shoe {
                size: 10,
                style: String::from("boot"),
            },
        ];

        let in_my_size = shoes_in_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe {
                    size: 10,
                    style: String::from("sneaker")
                },
                Shoe {
                    size: 10,
                    style: String::from("boot")
                },
            ]
        );
    }
}
```

## Comparing Performance: Loops vs. Iterators

Iterators may not be slower than writing low level `for` loops, because they get compiled down to roughly the same code. Iterator sare one of Rust's _zero-cost abstractions_, meaning the abstraction imposes no additional runtime overhead.
