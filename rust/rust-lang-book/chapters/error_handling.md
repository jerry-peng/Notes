<!-- markdownlint-disable MD013 -->

# Error Handling

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Error Handling](#error-handling)
  - [Unrecoverable Errors with `panic!`](#unrecoverable-errors-with-panic)
    - [Unwinding stack or Aborting in Response to a Panic](#unwinding-stack-or-aborting-in-response-to-a-panic)
    - [Using a `panic!` Backtrace](#using-a-panic-backtrace)
  - [Recoverable Errors with `Result`](#recoverable-errors-with-result)
    - [Matching on Different Errors](#matching-on-different-errors)
    - [Shortcuts for Panic on Error: `unwrap` and `expect`](#shortcuts-for-panic-on-error-unwrap-and-expect)
    - [Propagating Errors](#propagating-errors)
    - [A Shortcut for Propagating Errors](#a-shortcut-for-propagating-errors)
    - [Where The Error Propagation Shortcut Operator Can Be Used](#where-the-error-propagation-shortcut-operator-can-be-used)
  - [To `panic!` or Not to `panic!`](#to-panic-or-not-to-panic)
    - [Examples, Prototype Code, and Tests](#examples-prototype-code-and-tests)
    - [Cases in Which You Have More Information Than the Compiler](#cases-in-which-you-have-more-information-than-the-compiler)
    - [Guidelines for Error Handling](#guidelines-for-error-handling)
    - [Creating Custom Types for Validation](#creating-custom-types-for-validation)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Unrecoverable Errors with `panic!`

```rust
fn main() {
    panic!("crash and burn");
}
```

### Unwinding stack or Aborting in Response to a Panic

By default, when a panic occurs, program starts _unwinding_, which means Rust walks back up the stack and clean data from each function in stack, which is a lot of work.
Alternatively, the program can immediately _abort_, which ends program without cleaning up, and leave the memory for operating system to clean up.

If we need to make binary as small as possible, we can switch to abort adding configuration to `Cargo.toml`:

```toml
[profile.release]
panic = 'abort'
```

### Using a `panic!` Backtrace

```rust
fn main() {
    let v = vec![1, 2, 3];
    v[99];
}
```

Outputs:

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Outputs with backtrace:

```text
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/std/src/panicking.rs:483
   1: core::panicking::panic_fmt
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/panicking.rs:85
   2: core::panicking::panic_bounds_check
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/panicking.rs:62
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/slice/index.rs:255
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/slice/index.rs:15
   5: <alloc::vec::Vec<T> as core::ops::index::Index<I>>::index
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/alloc/src/vec.rs:1982
   6: panic::main
             at ./src/main.rs:4
   7: core::ops::function::FnOnce::call_once
             at /rustc/7eac88abb2e57e752f3302f02be5f3ce3d7adfb4/library/core/src/ops/function.rs:227
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

In order to get backtraces, debug symbols must be enabled. Debug symbols are enabled by default when using `cargo build` or `cargo run` without the `--release` flag.

## Recoverable Errors with `Result`

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` and `E` are generic type parameters, which will be discussed in Chapter 10. `Result` is included in prelude.

`Result` is returned when a method can fail.

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```

The code will panic when a file cannot be opened, due to non-existent file, or permission issues.

### Matching on Different Errors

The code above will fail no matter why `File::open` failed. What we want to do instead is take different actions for different reasons:

- If `File::open` failed because file does not exist, we create the file and return handle to new file.
- If `File::open` failed for other reasons, let program panic.

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error)
            }
        },
    };
}
```

In Chapter 13, we'll learn about closures; the `Result<T, E>` type has many methods that accept a closure and are implemented using `match` expressions.
Using those methods will make the code more concise. There is a more advanced way of writing logic in above program.

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

This code does not have `match` and is cleaner.

### Shortcuts for Panic on Error: `unwrap` and `expect`

`unwrap` method returns value inside `Ok`, or panics if `Result` is `Err` variant.

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

`expect` is similar to `unwrap` and let us choose `panic!` error message.

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

### Propagating Errors

A function that returns errors to the calling code using `match`

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

### A Shortcut for Propagating Errors

Code below has same functionality as code above.

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

The `?` after `Result` value is defined to work the same way as the `match` expressions handling `Result`. If value of `Result` is an `Ok`, the value inside `Ok` will get returned from expression,
and the program will continue. If the value is an `Err`, the `Err` will be returned from the whole function.

There is a difference between what the `match` expression handling `Result` versus `?` operator. Error values that have `?` operator called on go through `from` function,
defined in `From` trait in standard library, which is used to convert errors from one type to another. So when `?` calls `from` function, error type received is converted
into the error type defined in return type of current function.

As long as there's an `impl From<ReceivedError> for ReturnedError` to define conversion from `ReceivedError` to `ReturnedError` in trait's `from` function,
the `?` operator takes care of the `from` function automatically.

We can simplify code above further by chaining method calls immediately after `?`.

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

Another way to write above code is to use built-in function to read file to string.

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

### Where The Error Propagation Shortcut Operator Can Be Used

`?` operator can only be used in function that have a return type compatible with the value the `?` is used on. The return type of function has to be a `Result` to be compatible with `?` `Err` return.

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
// Compile error: cannot use the `?` operator in a function that returns `()`
```

To fix compilation error, there are two options.

1. Change return type to be `Result<T, E>` if there are no restrictions preventing it.
1. Use `match` or one of the methods to handle `Result<T, E>` when appropriate.

`?` can also be used on `Option<T>`. As with `Result`, `?` can only be used on `Option` in a function that returns an `Option`. If the value is is `None`, `None` will be returned early
from the function, otherwise, the value inside `Some` is the resulting value and function continues.

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

`lines` method returns an iterator over the lines in the string. `next` call on iterator gives the first value from iterator, which is the first line of text.
If `text` is an empty string, call to `next` will return `None`, and since we use `?`, the function will stop and return `None`.
If `text` is not an empty string, `?` extracts the string slice, and calling `chars` method on string slice returns iterator over its characters. Calling `last` on the iterator returns
the last item in the iterator. This is an `Option` because it's possible the first line is an empty string, e.g. `"\nhi"`. If there is a last character on the first line,
`Some` variant will be returned.

`?` does not convert between `Result` and `Option`. In those cases, you can use methods like `ok` method on `Result` or the `ok_or` method on `Option` to do the conversion.

All `main()` we've used return `()`. `main` is special because it's the entry and exit point of executable programs, and there are restrictions on return type.
Luckily, `main` can also return `Result<(), E>`, for the `main` example code above, we can change return type to `Result<(), Box<dyn Error>>` and add a return value `Ok(())` at the end:

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())
}
```

The `Box<dyn Error>` type is a trait object, which will be explained in _Using Trait Objects that Allow for Values of Different Types_ section in
_Object Oriented Programming Features of Rust_ chapter. For now, we can read `Box<dyn Error>` as "any kind of error".

When `main` function returns `Result<(), E>`, the executable will exit with 0 if `main` returns `Ok(())` and will exit with nonzero value if `main` returns an `Err` value.

`main` function may return any types that implement `std::process::Termination` trait, but as the time this note was written, `Termination` trait is only available in Nightly Rust.

## To `panic!` or Not to `panic!`

When calling `panic!`, it means we've determined the situation is unrecoverable on behalf of calling code. When choosing to return `Result`,
we give calling code options and allow calling code to recover or panic if `Result` is `Err`.

Returning `Result` is a good default. In situations such as examples, prototype code, and tests, it's more appropriate to write code that panics than returning `Result`.

### Examples, Prototype Code, and Tests

In examples, it's understood that a call to method like `unwrap` that could panic is meant as a placeholder.

Similarly, `unwrap` and `expect` methods are very handy during prototyping before we're ready to decide on how to handle errors. They leave clear markers for when you're ready to make the program more robust.

If a method call fails in a test, the whole test should fail. Because `panic!` is how a test is marked as a failure, calling `unwrap` and `expect` is exactly what should happen.

### Cases in Which You Have More Information Than the Compiler

It would be appropriate to call `unwrap` when you have some logic that ensures `Result` will have `Ok` value, but the logic is not understood by compiler. `Result` in this case will never be `Err` variant,
so it's perfectly acceptable to call `unwrap`.

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1".parse().unwrap();
```

`127.0.0.1` is a valid hard-coded IP address, but it does not change the return type of `parse` method (`Result`), which the compiler will make us handle even if the `Result` will never be `Err` variant.

### Guidelines for Error Handling

It's advisable to make our code panic when it's possible that your code could end up in a bad state. In this context, a _bad state_ is when some assumption, guarantee, contract, or invariant has been broken,
such as invalid values, contradictory values, or missing values are passed to code -- plus one or more of the following:

- The bad state is something that is unexpected, as opposed to something that will likely happen occasionally, like a user entering data in the wrong format.
- The code after this point needs to rely on not being in this bad state, rather than checking for the problem at every step.
- There is no good way to encode this informationin the types used. We'll work through an example of what we mean in the _Encoding States and Behavior as Types_ section in _Object Oriented Programming Features of Rust_ chapter.

If our code receive values that don't make sense, the best choice is to call `panic!`. Similarly, `panic!` is appropriate if we're calling external code that's out of our control and returns an invalid state that we
have no way of fixing.

If failure is expected, it's better to return `Result` than to make a `panic!` call. For example, HTTP request returning a status that indicates we hit rate limit.

When code performs operations on values, values should be verified to be valid first, and panic if values are invalid, since operating on invalid data can expose code to vulnerabilities.
Functions often have _contracts_; their behavior is only guaranteed if the inputs meet particular requirements. Panicking when contract is violated makes sense because it always indicates a caller-side bug.
Contract for a function, especially when a violation will cause panic, should be explained in API doc for the function.

However, having lots of error checks would be verbose and annoying. Fortunately, we can use Rust's type system to do many checks for us. For example, having a type rather than an `Option` guarantees
there is something rather than _nothing_, and we don't need to handle `Some` and `None` variants, and code trying to pass nothing to function won't even compile. Another example is using unsigned integer type such
as `u32` which ensures value is never negative.

### Creating Custom Types for Validation

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```

`Guess` struct has a `value` field that holds `i32`. The implemented `new` function that return instances of `Guess` validates the `value` passed in is within `[1, 100]`.
