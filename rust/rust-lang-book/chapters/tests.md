<!-- markdownlint-disable MD013 -->

# Writing Automated Tests

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Writing Automated Tests](#writing-automated-tests)
  - [How to Write Tests](#how-to-write-tests)
    - [Checking Results with `assert!` Macro](#checking-results-with-assert-macro)
    - [Testing Equality with Macros](#testing-equality-with-macros)
    - [Adding Custom Failure Messages](#adding-custom-failure-messages)
    - [Checking for Panics with Macros](#checking-for-panics-with-macros)
    - [Using `Result<T, E>` in Tests](#using-resultt-e-in-tests)
  - [Controlling How Tests Are Run](#controlling-how-tests-are-run)
    - [Running Tests in Parallel or Consecutively](#running-tests-in-parallel-or-consecutively)
    - [Showing Function Output](#showing-function-output)
    - [Running a Subset of Tests by Name](#running-a-subset-of-tests-by-name)
      - [Running Single Tests](#running-single-tests)
      - [Filtering to Run Multiple Tests](#filtering-to-run-multiple-tests)
    - [Ignoring Some Tests Unless Specifically Requested](#ignoring-some-tests-unless-specifically-requested)
  - [Test Organization](#test-organization)
    - [Unit Tests](#unit-tests)
      - [The Tests Module and `#[cfg(test)]`](#the-tests-module-and-cfgtest)
      - [Testing Private Functions](#testing-private-functions)
    - [Integration Tests](#integration-tests)
      - [The `tests` Directory](#the-tests-directory)
      - [Submodules in Integration Tests](#submodules-in-integration-tests)
      - [Integration Tests for Binary Crates](#integration-tests-for-binary-crates)
<!--toc:end-->

<!-- prettier-ignore-end -->

## How to Write Tests

At its simplest, a test in Rust is a function that's annotated with `test` attribute. Attribute are metadata about pieces of Rust code; one example is `derive` attribute used with structs.

To change a function into a test function, add `#[test]` on the line before `fn`. Run tests with `cargo test`, which builds a test runner library that runs functions annotated with `test` attributes
and reports on whether tests pass or fail.

When making a new library with Cargo, a test module with a test function in it is automatically generated for us.

```text
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

The contents of `src/lib.rs` file in `adder` library should look like this:

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Running the test produce outputs:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.57s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Because we don't have tests marked as ignored, the summary shows `0 ignored`. Since no filters are applied to tests, the summary shows `0 filtered out`.
The `0 measured` statistics is for benchmark tests that measure performance. Benchmark tests are available in nightly Rust as of the writing of this note.

`Doc-tests adder` is for the results of any documentation tests. We don't have documentation tests yet, but Rust can compile any code examples that appear in our API documentation.
This feature helps us keep docs and code in sync. Writing documentation tests are discussed in _Documentation Comments as Tests_ section in _More about Cargo and Creates.io_ chapter.

An example when test fails:

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```

Tests fail when something in the test function panics. Each test is run in a new thread, and when main thread sees that a test thread has died, the test is marked as failed. The test output:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.72s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::another ... FAILED
test tests::exploration ... ok

failures:

---- tests::another stdout ----
thread 'main' panicked at 'Make this test fail', src/lib.rs:10:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```

### Checking Results with `assert!` Macro

The `assert!` macro is provided by standard library. If `assert!` evaluates to true, the test passes, otherwise `assert!` macro calls the `panic!` macro and test fails.

Example struct code:

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(!smaller.can_hold(&larger));
    }
}
```

### Testing Equality with Macros

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

### Adding Custom Failure Messages

Custom messages to be printed with failure messages can be added as optional arguments to the `assert!`, `assert_eq!` and `assert_ne!` macros.
Any arguments specified after the one required argument to `assert!` or the two required arguments to `assert_eq!` and `assert_ne` are passed along to `format!` macro,
so we can pass format string containing `{}` placeholders and values going into those placeholders.

```rust
// Buggy function
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            "Greeting did not contain name, value was `{}`",
            result
        );
    }
}
```

Test output:

```text
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.93s
     Running unittests (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread 'main' panicked at 'Greeting did not contain name, value was `Hello!`', src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```

### Checking for Panics with Macros

To indicate a test should panic, add `#[should_panic]` attribute after `#[test]` attribute and before the test function it applies to.

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
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

To make `should_panic` tests more precise, we can add an optional `expected` parameter to the `should_panic` attribute.

```rust
// --snip--
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

The test will pass because the value we put in the `should_panic` attribute's `expected` parameter is a substring of the message that the `Guess::new` function panics with.

### Using `Result<T, E>` in Tests

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

Writing tests so they return `Result<T, E>` enables us to use the question mark operator in the body of tests, which can be a convenient way to write tests that should fail if any operation
within them return an `Err` variant.

`#[should_panic]` can't be used on tests that use `Result<T, E>`. Instead, return `Err` value directly when test should fail.

## Controlling How Tests Are Run

We can specify command line options to change the default behavior of `cargo test`. Some command line options go to `cargo test`, and some go to the resulting test binary.
To separate these two types of arguments, we can list arguments that go to `cargo test` followed by the separator `--` and then the ones that go to the test binary.

`cargo test --help` displays the options we can use with `cargo test`, and `cargo test -- --help` displays the options we can use after separator `--`

### Running Tests in Parallel or Consecutively

To run tests in parallel, or if we want more fine-grained control over the number of threads used, we can send the `--test-threads` flag and the number of threads to use on the test binary.

```sh
cargo test -- --test-threads=1
```

### Showing Function Output

By default, if a test passes, Rust's test library captures anything printed to standard output.

```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

Test output (note `println` output is not printed for the passed test):

```text
$ cargo test
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread 'main' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```

To show output of successful tests, use `--show-output` flag

```sh
cargo test -- --show-output
```

### Running a Subset of Tests by Name

Example code:

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

#### Running Single Tests

```text
$ cargo test one_hundred
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.69s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s
```

The test output let us know we have more tests than what was run by displaying `2 filtered out` at the end of summary.

#### Filtering to Run Multiple Tests

We can specify part of a test name, and any test whose name matches that value will be run.

```text
$ cargo test add
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s
```

Note module in which a test appears becomes part of the test's name, so we can run all the tests in a module by filtering on the module's name.

### Ignoring Some Tests Unless Specifically Requested

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

Test output:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The `expensive_test` function is listed as `ignored`. If we want to run only the ignored tests, we can use `cargo test -- --ignored`:

```text
$ cargo test -- --ignored
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

## Test Organization

Rust community thinks about tests in term of two main categories: _unit tests_ and _integration tests_. Unit tests are small and more focused, testing one module in isolation at a time,
and can test private interfaces. Integration tests are entirely external to our library and use code the same way any other external code would, using only public interface and
potentially exercising multiple modules per test.

### Unit Tests

Unit tests are conventionally put in the `src` directory in each file with the code that they're testing, and test modules are conventionally named `tests` in each file to contain
the test functions and to annotate the module with `cfg(test)`.

#### The Tests Module and `#[cfg(test)]`

`#[cfg(test)]` annotation on the tests module tells Rust to compile and run the test code only when we run `cargo test`, not when we run `cargo build`, which saves compile time
and space when building library because tests are not included.
Integration tests go into a different directory, so they don't need the `#[cfg(test)]` annotation.

#### Testing Private Functions

There's debate within testing community on whether or not private functions should be tested directly, and other languages make it difficult or impossible to test private functions.

Regardless of testing ideology, Rust's privacy rules allow testing private functions.

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

### Integration Tests

#### The `tests` Directory

`tests` directory are created at the top level of our project directory, next to `src`. Cargo knows to look for integration test files in this directory.
We can then make as many test files as we want in this directory, and Cargo will compile each of the files as an individual crate.

```rust
// Filename: tests/integration_test.rs
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

We add `use adder` at the top of the code unlike in unit tests, because each file in the `tests` directory is a separate crate, so we need to bring library into each test's crate scope.

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 1.31s
     Running unittests (target/debug/deps/adder-1082c4b063a8fbe6)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-1082c4b063a8fbe6)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The three sections of the output include the unit tests, the integration tests and the doc tests.

Each integration test file has its own section, so if more files are added in the `tests` directory, there will be more integration test sections.

We can still run a particular integration test function by specifying test function's name as argument to `cargo test`. To run all tests of a particular integration test file,
use the `--test` argument.

```text
$ cargo test --test integration_test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.64s
     Running tests/integration_test.rs (target/debug/deps/integration_test-82e7799c1bc62298)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

This command runs only the tests in `tests/integration_test.rs` file.

#### Submodules in Integration Tests

To have common helper functions in integration tests, we can create `tests/common.rs` and place a function named `setup` in it. However, when running integration tests, there will be
a section in the test output for the _common.rs_ file, even though this file doesn't contain any test functions nor did we call the `setup` function from anywhere.

To fix this issue, instead of creating `tests/common.rs`, create `tests/common/mod.rs`, which is an alternate naming convention that Rust also understands.
Naming file this way tells Rust not to treat the `common` module as an integration test file. Files in subdirectories of the `tests` directory don't get compiled as separate crates
or have sections in the test output.

To use the `common` module:

```rust
// Filename: tests/integration_test.rs

use adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

#### Integration Tests for Binary Crates

If a project is a binary crate and only contains `src/main.rs` and does not have a `src/lib.rs` file, we can't create integration tests in the `tests` directory and bring functions
defined in `src/main.rs` file into scope using `use` statement. Only library crates expose functions that other crates can use.

This is one of the reasons Rust projects that provide a binary have a straightforward `src/main.rs` file that calls logic that live in the `src/lib.rs` file, which makes the library testable with
integration tests.
