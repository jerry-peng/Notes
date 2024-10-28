<!-- markdownlint-disable MD013 -->

# Smart Pointers

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Smart Pointers](#smart-pointers)
  - [Using `Box` to Point to Data on the Heap](#using-box-to-point-to-data-on-the-heap)
    - [Using a `Box` to Store Data on the Heap](#using-a-box-to-store-data-on-the-heap)
    - [Enabling Recursive Types with Boxes](#enabling-recursive-types-with-boxes)
      - [More information About the Cons List](#more-information-about-the-cons-list)
      - [Using `Box` to Get a Recursive Type with a Known Size](#using-box-to-get-a-recursive-type-with-a-known-size)
  - [Treating Smart Pointers Like Regular References with the `Deref` Trait](#treating-smart-pointers-like-regular-references-with-the-deref-trait)
    - [Following the Pointer to the Value](#following-the-pointer-to-the-value)
    - [Using `Box` Like a Reference](#using-box-like-a-reference)
    - [Defining Our Own Smart Pointer](#defining-our-own-smart-pointer)
    - [Treating a Type Like a Reference by Implementing the `Deref` Trait](#treating-a-type-like-a-reference-by-implementing-the-deref-trait)
    - [Implicit Deref Coercions with Functions and Methods](#implicit-deref-coercions-with-functions-and-methods)
    - [How Deref Coercion Interacts with Mutability](#how-deref-coercion-interacts-with-mutability)
  - [Running Code on Cleanup with the `Drop` Trait](#running-code-on-cleanup-with-the-drop-trait)
    - [Dropping a Value Early with `std::mem::drop`](#dropping-a-value-early-with-stdmemdrop)
  - [`Rc`, the Reference Counted Smart Pointer](#rc-the-reference-counted-smart-pointer)
    - [Using `Rc` to Share Data](#using-rc-to-share-data)
    - [Cloning an `Rc` Increases the Reference Count](#cloning-an-rc-increases-the-reference-count)
  - [`RefCell` and the Interior Mutability Pattern](#refcell-and-the-interior-mutability-pattern)
    - [Enforcing Borrowing Rules at Runtime with `RefCell`](#enforcing-borrowing-rules-at-runtime-with-refcell)
    - [Interior Mutabilituy: A Mutable Borrow to an Immutable Value](#interior-mutabilituy-a-mutable-borrow-to-an-immutable-value)
      - [A Use Case for Interior Mutability: Mock Objects](#a-use-case-for-interior-mutability-mock-objects)
      - [Keeping Track of borrows at Runtime with `RefCell`](#keeping-track-of-borrows-at-runtime-with-refcell)
    - [Having Multiple Owners of Mutable Data by Combining `Rc` and `RefCell`](#having-multiple-owners-of-mutable-data-by-combining-rc-and-refcell)
  - [Reference Cycles Can Leak Memory](#reference-cycles-can-leak-memory)
    - [Creating a Reference Cycle](#creating-a-reference-cycle)
    - [Preventing Reference Cycles: Turning an `Rc` into a `Weak`](#preventing-reference-cycles-turning-an-rc-into-a-weak)
      - [Creating a Tree Data Structure: a Node with Child Nodes](#creating-a-tree-data-structure-a-node-with-child-nodes)
      - [Adding a Reference from a Child to its Parent](#adding-a-reference-from-a-child-to-its-parent)
      - [Visualizing Changes to strong and weak counts](#visualizing-changes-to-strong-and-weak-counts)
<!--toc:end-->

<!-- prettier-ignore-end -->

Smart pointers are data structures that act like a pointer but also have additional metadata and capabilities. We have already encountered a few smart pointers including `String` and `Vec<T>`.

Smart pointers are structs that implement the `Deref` and `Drop` traits. The `Deref` trait allows an instance of the smart pointer struct to behave like a reference so you can write your code to work with either references or smart pointers. The `Drop` trait allows you to customize the code that’s run when an instance of the smart pointer goes out of scope.

This chapter will cover most common smart pointers:

- `Box<T>` for allocating values on the heap
- `Rc<T>`, a reference counting type that enables multiple ownership
- `Ref<T>` and `RefMut<T>`, accessed through `RefCell<T>`, a type that enforces the borrowing rules at runtime instead of compile time.

In addition, _interior mutability_ pattern and _reference cycles_ will be covered.

## Using `Box` to Point to Data on the Heap

Boxes allow us to store data on the heap rather than the stack, so that only the pointer to the heap data is stored on the stack. They don't have performance overhead but don't have extra capabilities either. They are often used in following cases:

- When we want to use a value of a type whose size cannot be known at compile time but exact size is required.
- When we want to transfer ownership of a large amount of data but ensure data won't be copied.
- When we want to own a value that implements a particular trait rather than being a specific type.

### Using a `Box` to Store Data on the Heap

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {b}");
}
```

Variable `b` has the value of a `Box` that points to the value `5` which is allocated on the heap. The program will print `b = 5`; we can access the data in the box similar to how we would if the data were on the stack. When a box goes out of scope, the box and the data it points to will be deallocated.

### Enabling Recursive Types with Boxes

A value of _recursive type_ can have another value of the same type as part of itself. (think of `Node` in a `Tree` structure) Recursive types pose an issue because at compile time Rust needs to know how much space a type takes up, and the nesting of values could theoretically continue infinitely. Box can help us since it has a known size.

To demonstrate, we'll explore the _cons list_, a data type commonly found in functional programming languages.

#### More information About the Cons List

A _cons list_ is a data structure that comes from the Lisp programming language and is made up of nested pairs, and is the Lisp version of linked list. For the sake of simplicity, we'll implement the data structure to hold `i32` values.

```rust
enum List {
    Cons(i32, List),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

If we try to compile the code, it will error:

```text
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^
2 |     Cons(i32, List),
  |               ---- recursive without indirection
  |
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +

error[E0391]: cycle detected when computing when `List` needs drop
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^
  |
  = note: ...which immediately requires computing when `List` needs drop again
  = note: cycle used when computing whether `List` needs drop
  = note: see https://rustc-dev-guide.rust-lang.org/overview.html#queries and https://rustc-dev-guide.rust-lang.org/query.html for more information

Some errors have detailed explanations: E0072, E0391.
For more information about an error, try `rustc --explain E0072`.
error: could not compile `cons-list` (bin "cons-list") due to 2 previous errors
```

The compiler cannot figure out how much space is needed to store a `List` value, and it's suggesting us to use `Box` instead.

#### Using `Box` to Get a Recursive Type with a Known Size

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

For `List` type, the `Nil` variant stores no value, and `List` variant will take up the size of `i32` plus the box's pointer data. We've now decomposed the infinite recursive chain of types into something of which the compiler can calculate the size.

The `Box<T>` type is a smart pointer because it implements the `Deref` trait which allows it to be treated like references. When a `Box<T>` value goes out of scope, the heap data pointed by the box is cleaned up as well because of the `Drop` trait implementation.

## Treating Smart Pointers Like Regular References with the `Deref` Trait

Implementing the `Deref` trait allows you to customize the behavior of the _dereference operator_ `*`.

### Following the Pointer to the Value

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

This is similar to C++. If we tried to write `assert_eq!(5, y);`, we would get this compilation error:

```text
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0277]: can't compare `{integer}` with `&{integer}`
 --> src/main.rs:6:5
  |
6 |     assert_eq!(5, y);
  |     ^^^^^^^^^^^^^^^^ no implementation for `{integer} == &{integer}`
  |
  = help: the trait `PartialEq<&{integer}>` is not implemented for `{integer}`
  = note: this error originates in the macro `assert_eq` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0277`.
error: could not compile `deref-example` (bin "deref-example") due to 1 previous error
```

### Using `Box` Like a Reference

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

The main difference from the previous section is that here we set `y` to be an instance of `Box<T>` pointing to a copied value of x rather than a reference pointing to value of `x`.

### Defining Our Own Smart Pointer

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

The code will result in compilation error:

```text
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0614]: type `MyBox<{integer}>` cannot be dereferenced
  --> src/main.rs:14:19
   |
14 |     assert_eq!(5, *y);
   |                   ^^

For more information about this error, try `rustc --explain E0614`.
error: could not compile `deref-example` (bin "deref-example") due to 1 previous error
```

Our `MyBox<T>` type can't be dereferenced because we have not implemented `Deref` trait.

### Treating a Type Like a Reference by Implementing the `Deref` Trait

The `Deref` trait, provided by the standard library, requires us to implement one method named `deref` that borrows `self` and returns a reference to the inner data.

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}
```

The `type Target = T;` syntax defines an associated type for the `Deref` trait to use. Associated types are a slightly different way of declaring a generic parameter, which will be discussed in detail in Chapter 19.

In the example above, `deref` method returns reference to the value we want to access with the `*` operator. Now the main function in previous section would compile.

Without `Deref` trait, the compiler can only dereference `&` references. In previous section's code `*y` actually runs this code behind the scene:

```rust
*(y.deref())
```

Rust substitutes the `*` operator with a call to the `deref` method exactly once, and then a plain dereference so we don't have to think about whether or not we need to call the `deref` method. This feature lets us write code that function identically whether we have a regular reference or a type that implement `Deref`.

The reason `deref` method returns a reference to a value, and plain dereference outside parentheses in `*(y.deref())` is still necessary, is because the alternative would move the value out o f`self`.

### Implicit Deref Coercions with Functions and Methods

_Deref coercion_ converts a reference to a type that implements the `Deref` trait into a reference to another type. For example, deref coercion can convert `&String` to `&str&` because `String` implements the `Deref` trait such that it returns `&str`.

Deref coercion was added so programmers writing functions and method calls don't need to add as many explicit references and dereferences with `&` and `*`. It also lets us write code that can work for either references or smart pointers.

Here is an example function that takes a `&str` parameter:

```rust
fn hello(name: &str) {
    println!("Hello, {name}!");
}
```

Below is comparison between function calls with and without deref coercion:

```rust
// With deref coercion
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

```rust
// Without deref coercion
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

Code without deref coercion is harder to read and write.

When the `Deref` trait is defined for the types involved, Rust will analyze the types and use `Deref::deref` as many times as necessary to get a reference to match the parameter's type. The number of times that `Deref::deref` needs to be inserted is resolved at compile time, so no runtime penalty.

### How Deref Coercion Interacts with Mutability

Similar to how `Deref` trait is used to override `*` operator on immutable references, for mutable references, `DerefMut` trait can be used.

Rust does deref coercion when it finds types and trait implementations in three cases:

- From `&T` to `&U` when `T:Deref<Target=U>`
- From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
- From `&mut T` to `&U` when `T: Deref<Target=U>`

First case states that if you have `&T` and `T` implements `Deref` to type `U`, you can get a `&U` transparently. Second case is the same except second implements mutability.

Third case is trickier: Rust can coerce a mutable reference to an immutable one, but the reverse is not possible due to borrowing rules; if you have a mutable reference, that mutable reference must be the only reference to that data. Converting an immutable reference to a mutable reference would require that the initial immutable reference is the only immutable reference to the data, but borrowing rules does not guarantee that.

## Running Code on Cleanup with the `Drop` Trait

The `Drop` trait allows us to customize what happens when a value is about to go out of scope. It can be used to release resources like files or network connections.

In Rust, you can specify some code to be run whenever a value goes out of scope by implementing the `Drop` trait, and compile will insert this code automatically. As a result, we don't need to be careful about placing cleanup code. The `Drop` trait requires implementing `drop` method that takes a mutable reference to `self`.

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}
```

Program output:

```text
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.60s
     Running `target/debug/drop-example`
CustomSmartPointers created.
Dropping CustomSmartPointer with data `other stuff`!
Dropping CustomSmartPointer with data `my stuff`!
```

The `Drop` trait is included in the prelude, so we don't need to bring it into scope. The `drop` method is implicitly called when `CustomSmartPointer` goes out of scope. Variables are dropped in the reverse order of creation, so `d` was dropped before `c`.

### Dropping a Value Early with `std::mem::drop`

Occasionally, we may want to clean up a value early. For instance, when using smart pointers that manager locks, we might want to force the `drop` method that releases the lock so other code in same scope can acquire the lock. Rust does not allow calling `Drop` trait's `drop` method manually; instead we have to call the `std::mem::drop` function provided by the standard library if we want to force a value to be dropped.

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    c.drop();
    println!("CustomSmartPointer dropped before the end of main.");
}
```

This code will cause compiler error:

```text
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
error[E0040]: explicit use of destructor method
  --> src/main.rs:16:7
   |
16 |     c.drop();
   |     --^^^^--
   |     | |
   |     | explicit destructor calls not allowed
   |     help: consider using `drop` function: `drop(c)`

For more information about this error, try `rustc --explain E0040`.
error: could not compile `drop-example` due to previous error
```

Rust disallows explicit `drop` call because Rust would still automatically call `drop` on value when it goes out of scope, which would cause a double free.

To manually drop a value, we use `std::mem::drop`. The function can be called by passing the value we want to force drop as an argument. The function is in the prelude.

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```

Program output:

```text
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.73s
     Running `target/debug/drop-example`
CustomSmartPointer created.
Dropping CustomSmartPointer with data `some data`!
CustomSmartPointer dropped before the end of main.
```

The `Drop` trait allows us to make cleanup convenient and safe. For instance, we could use it to create our own memory allocator. We also don't have to worry about accidentally cleaning up values still in use, as the ownership system ensures `drop` gets called only once when the value is no longer being used.

## `Rc`, the Reference Counted Smart Pointer

There are cases when a single value may have multiple owners. For example, in graph data structures, multiple edges might point to the same node, and the node is conceptually owned by all of adjustcent edges.

Multiple ownership can be enabled explicitly by using Rust type `Rc<T>`, abbreviation for _reference counting_. The type keeps track of number of references to a value to determine whether or not the value is still in use. If there are zero references to the value, the value can be cleaned up.

Note that `Rc<T>` is only for use in single-threaded contexts. Reference counting in multithreaded will be discussed Chapter 16.

### Using `Rc` to Share Data

Using cons list, we'll try creating a scenario where two `List` instances both point to the same `List` instance using `Box<T>`.

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
```

The compiler would throw error:

```text
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0382]: use of moved value: `a`
  --> src/main.rs:11:30
   |
9  |     let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
   |         - move occurs because `a` has type `List`, which does not implement the `Copy` trait
10 |     let b = Cons(3, Box::new(a));
   |                              - value moved here
11 |     let c = Cons(4, Box::new(a));
   |                              ^ value used here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `cons-list` due to previous error
```

To fix this, we need to use `Rc<T>`:

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

When we create `b`, instead of taking ownership of `s`, we'll clone the `Rc<List>` that `a` is holding, thereby increasing the number of references from 1 to 2 and letting `a` and `b` share ownership of the data in the `Rc<List>`.

The `Rc<T>` type is not inthe prelude so we need to bring it into scope with a `use` statement.

We could have called `a.clone()` rather than `Rc::clone(&a)`, but Rust's convention is to use `Rc::clone` in this case, because it does not make a deep copy of all the underlying data, instead it increments the reference count, which is a lot cheaper when compared to deep copying. By using `Rc::clone` makes it easier to visually recognize cloning `Rc` is not deep copy.

### Cloning an `Rc` Increases the Reference Count

Use `Rc::Strong_count()` to print reference count. It's not named `count` because there is also a `weak_count` that will be discussed later in the chapter.

```rust
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
```

```text
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.45s
     Running `target/debug/cons-list`
count after creating a = 1
count after creating b = 2
count after creating c = 3
count after c goes out of scope = 2
```

When `b` and then `a` go out of scope at the end of `main`, the count is decreased to 0 and the `Rc<List>` is cleaned up completely. The `Rc<T>` reference count ensures value remains valid as long as any of the owners still exist.

## `RefCell` and the Interior Mutability Pattern

_Interior mutability_ is a design pattern in Rust that allows us to mutate data even if there are immutable references. This is disallowed by the borrowing rule normally, but the pattern uses `unsafe` code inside a data structure to bend the borrowing rule. Unsafe code indicates to compiler that we are checking the rules manually instead of relying on the compiler.

We can use types that use interior mutability pattern only when we can ensure that the borrowing rules will be followed at runtime. The `unsafe` code involved is wrapped in a safe API, and outer type is still immutable.

### Enforcing Borrowing Rules at Runtime with `RefCell`

REcall borrowing rules:

- We can either have 1 mutable reference or any number of immutable references.
- References must always be valid.

With references and `Box<T>`, borrowing rules are enforced at compile time, and we get compiler errors if rules are broken. With `RefCell<T>`, rules are enforced at runtime, and program panic if rules are broken.

The advantages of checking borrowing rules at compile time is errors are caught sooner in the development process, and no impact on runtime performance because all analysis is completed beforehand, which is why this is Rust's default behavior.

However, Rust compiler is inherently conservative, meaning certain memory-safe scenarios are disallowed by the compile-time checks, and some properties of code are impossible to detect by analyzing the code; most famous example is the the Halting Problem. The `RefCell<T>` type is useful when we are sure the code follow the borrowing rules but compiler cannot understand and guarantee that.

Similar to `Rc<T>`, `RefCell<T>` is only for single-threaded scenarios.

Recap of reasons to choose between `Box<T>`, `Rc<T>`, or `RefCell<T>`:

- Owner
  - `Rc<T>` enables multiple owners of the same data.
  - `Box<T>` and `RefCell<T>` have single owners.
- Borrows
  - `Box<T>` allows immutable or mutable borrows checked at compile time.
  - `Rc<T>` allows only immutable borrows checked at compile time.
  - `RefCell<T>` allows immutable or mutable borrows checked at runtime.
- `RefCell<T>` allows mutable borrows checked at runtime, so we can mutate value within even if `RefCell<T>` itself is immutable.

### Interior Mutabilituy: A Mutable Borrow to an Immutable Value

There are situations in which it would be useful for a value to mutate itself in its methods, but appear immutable to other code. Using `RefCell<T>` is one way to get this interior mutability, but it does not get around borrowing rules completely, because it is checked at runtime, and if rules are violated, the program panics.

#### A Use Case for Interior Mutability: Mock Objects

Below is an example that we will create mock objects for:

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}
```

In unit test, we'll create a `MockMessenger` that implements the `Messenger` trait which can be used to verify `LimitTracker` behavior.

```rust
#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: vec![],
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

However, the compiler throws error:

```text
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
error[E0596]: cannot borrow `self.sent_messages` as mutable, as it is behind a `&` reference
  --> src/lib.rs:58:13
   |
2  |     fn send(&self, msg: &str);
   |             ----- help: consider changing that to be a mutable reference: `&mut self`
...
58 |             self.sent_messages.push(String::from(message));
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `self` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `limit-tracker` due to previous error
warning: build failed, waiting for other jobs to finish...
```

We can't modify the `MockMessenger` to keep track of messages, because the `send` method takes an immutable reference to `self`. The compiler suggesion also would not work as using `&mut self` causes function signature to not match `Messenger` trait's definition.

To fix this, we can store `sent_messages` within a `RefCell<T>`:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

Note that we call `borrow_mut` on the `RefCell<Vec<String>>` in `self.sent_messages` to get mutable reference so we can call `push` on the mutable reference to the vector. During assetion, we call `borrow` to get an immutable reference to the vector.

#### Keeping Track of borrows at Runtime with `RefCell`

When creating immutable and mutable references, we use the `&` and `&mut` syntax. With `RefCell<T>`, we use the `borrow` and `borrow_mut` methods, which are part of the safe API that belongs to `RefCell<T>` The `borrow` method returns smart pointer type `Ref<T>` and `borrow_mut` returns smart pointer type `RefMut<T>`. Both types implement `Deref`, so we can treeat them like regular references.

The `RefCell<T>` keeps track of how many `Ref` and `RefMut` smart pointers are currently active. Every time we call `borrow`, `RefCell` increases count of active immutable borrows. When a `Ref` value goes out of scope, immutable borrow count decreases by 1.

Just like compile-time borrowing rule, `RefCell` lets us have many immutable borrows, or 1 mutable borrow at any point in time. If we violate these rules, the implementation of `RefCell` will panic at runtime. Below is a demonstration:

```rust
    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            let mut one_borrow = self.sent_messages.borrow_mut();
            let mut two_borrow = self.sent_messages.borrow_mut();

            one_borrow.push(String::from(message));
            two_borrow.push(String::from(message));
        }
    }
```

The implementation above would result in runtime panic:

```text
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
    Finished test [unoptimized + debuginfo] target(s) in 0.91s
     Running unittests src/lib.rs (target/debug/deps/limit_tracker-e599811fa246dbde)

running 1 test
test tests::it_sends_an_over_75_percent_warning_message ... FAILED

failures:

---- tests::it_sends_an_over_75_percent_warning_message stdout ----
thread 'tests::it_sends_an_over_75_percent_warning_message' panicked at 'already borrowed: BorrowMutError', src/lib.rs:60:53
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::it_sends_an_over_75_percent_warning_message

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
```

Catching borrowing errors at runtime means mistakes are caught later in the development process, possibly after code is deployed. Additionally, the code would incur a small runtime performance penalty as borrows are tracked at runtime.

### Having Multiple Owners of Mutable Data by Combining `Rc` and `RefCell`

A common way to use `RefCell<T>` is in combination with `Rc<T>`. If we have an `Rc` that holds a `RefCell`, you can get a value that can have multiple owners that can be mutated.

For instance, let's use the cons list example:

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

Program output:

```text
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.63s
     Running `target/debug/cons-list`
a after = Cons(RefCell { value: 15 }, Nil)
b after = Cons(RefCell { value: 3 }, Cons(RefCell { value: 15 }, Nil))
c after = Cons(RefCell { value: 4 }, Cons(RefCell { value: 15 }, Nil))
```

Note that `RefCell` does not work in multithreaded code. `Mutex<T>` is the thread-safe version of `ReCell<T>` which will be discussed in Chapter 16.

## Reference Cycles Can Leak Memory

Rust's memory safety guarantees make it difficult to accidentally create memory leaks, but not impossible. Preventing memory lekas entirely is not one of Rust's guarantees, meaning memory leaks are memory safe in Rust.

One possible way to create memory leak is reference cycles, where an items refer to each other in a cycle. This creates memory leaks because reference count of each item in the cycle will never reach 0, and are never dropped.

### Creating a Reference Cycle

Let's use cons list example:

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
```

Program output:

```text
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.53s
     Running `target/debug/cons-list`
a initial rc count = 1
a next item = Some(RefCell { value: Nil })
a rc count after b creation = 2
b initial rc count = 1
b next item = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
b rc count after changing a = 2
a rc count after changing a = 2
```

We can see that both `a` and `b` have reference count of 2 due to reference cycle. When both variables go out of scope, both reference counts would decrease to 1, but never 0, which means both references are not properly dropped.

If we try and uncomment the last `println!` in the code block above and run the program, Rust will try to print the reference cycle with `a` pointing to `b` pointing to `a` and so on until it overflows the stack.

In the example, consequence of reference cycle is not very dire, as program ends right after reference cycle is created. In the real world application, reference cycles can cause program to allocate lots of memory and hold on to it for a long time.

Creating reference cycles is not easily done but not impossible. When we have `RefCell<T>` values that contain `Rc<T>` values or similar nexted combinations of types with interior mutability and reference counting, we need to ensure no cycles are created using automated tests, code reviews, and other software development practices to minize, as Rust cannot be relied on.

Another solution to avoid reference cycles is reorganizing data structures so that some references express ownership and some references don't. As a result, you can have cycles made up of some ownership relationships and some non-ownership relationships, and only the former affect whether or not a value can be dropped. In the cons list example in code block above, we always want `Cons` variants to own their list, so reorganizing the data structure isn't possible.

### Preventing Reference Cycles: Turning an `Rc` into a `Weak`

We can create a _weak reference_ to the value within `RC<T>` instance by calling `Rc::downgrade` and passing a reference to the `Rc<T>`. Strong references are how we can share ownership of an `Rc<T>` instance. Weak references don't express an ownership relationship, and weak reference count does not affect when an `Rc<T>` instance is cleaned up. Weak references will be broken when strong reference count is 0, which means they won't cause a reference cycle.

When calling `Rc::downgrade`, a smart pointer of type `Weak<T>` is returned, and `weak_count` is increased by 1. To get the value `Weak<T>` is referencing, call `upgrade` method which returns an `Option<Rc<T>>`, because the value that `Weak<T>` references may have been dropped.

#### Creating a Tree Data Structure: a Node with Child Nodes

We want a `Node` to own its children, and we want to share that ownership with variables so we can access each `Node` in the tree directly. To do this, we'll define `Vec<T>` items to be values of type `Rc<Node>`. We also want to modify the children nodes, so we have a `RefCell<T>` around the `Vec<Rc<Node>>`.

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
```

#### Adding a Reference from a Child to its Parent

TO make child node aware of its parent, we need to add a `parent` field to our `Node` struct definition. The type can't be `Rc<T>` because that would create a reference cycle with `leaf.parent` pointing to `branch` and `branch.children` pointing to `leaf`.

Conceptually, a parent node should own child nodes; if a parent node is dropped, so should the child nodes, but when a child node is dropped, the parent node should still exist. This is a case for weak references.

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```

Program output:

```text
leaf parent = None
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

Lack of infinite output indicates no reference cycle is created.

#### Visualizing Changes to strong and weak counts

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
    }

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
}
```

1st print shows `leaf` has strong count of 1 and weak count of 0.
2nd print shows `branch` has strong count of 1 and weak count of 1.
3rd print shows `leaf` has strong count of 2, and weak count of 0.
4th print shows `leaf`'s' parent references nothing as it went out of scope.
5th print shows `leaf` has strong count of 1, and weak count of 0.
