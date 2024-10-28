<!-- markdownlint-disable MD013 -->

# Fearless Concurrency

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Fearless Concurrency](#fearless-concurrency)
  - [Using Threads to Run Code Simultaneously](#using-threads-to-run-code-simultaneously)
    - [Creating a New Thread with `spawn`](#creating-a-new-thread-with-spawn)
    - [Waiting for All Threads to Finish Using `join` Handles](#waiting-for-all-threads-to-finish-using-join-handles)
    - [Using `move` Closures with Threads](#using-move-closures-with-threads)
  - [Using Message Passing to Transfer Data Between Threads](#using-message-passing-to-transfer-data-between-threads)
    - [Channels and Ownership Transference](#channels-and-ownership-transference)
    - [Sending Multiple Values and Seeing the Receiver Waiting](#sending-multiple-values-and-seeing-the-receiver-waiting)
    - [Creating Multiple Producers by Cloning the Transmitter](#creating-multiple-producers-by-cloning-the-transmitter)
  - [Shared-State Concurrency](#shared-state-concurrency)
    - [Using Mutexes to Allow Access to Data from One Thread at a Time](#using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time)
      - [The API of `Mutex`](#the-api-of-mutex)
      - [Sharing a `Mutex` Between Multiple Threads](#sharing-a-mutex-between-multiple-threads)
      - [Multiple Ownership with Multiple Threads](#multiple-ownership-with-multiple-threads)
      - [Atomic Reference Counting with `Arc`](#atomic-reference-counting-with-arc)
    - [Similarities Between `RefCell`/`Rc` and `Mutex`/`Arc`](#similarities-between-refcellrc-and-mutexarc)
  - [Extensible Concurrency with the `Sync` and `Send` Traits](#extensible-concurrency-with-the-sync-and-send-traits)
    - [Allowing Transference of Ownership Between Threads with `Send`](#allowing-transference-of-ownership-between-threads-with-send)
    - [Allowing Access from Multiple Threads with `Sync`](#allowing-access-from-multiple-threads-with-sync)
    - [Implmmenting `Send` and `Sync` Manually is Unsafe](#implmmenting-send-and-sync-manually-is-unsafe)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Using Threads to Run Code Simultaneously

Threads allow a program process to have independent parts run simultaneously, but it can also lead to problems such as:

- Race codntions, where threads are accessing data in an inconsistent order
- Deadlocks, where two threads are waiting for each other, preventing both threads from continuing.
- Bugs that only happen in certain situations that are hard to reproduce and fix reliatbly.

Rust standard library uses a _1:1_ model of thread implementation, whereby a program uses one operating system thread per language thread.

### Creating a New Thread with `spawn`

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

Program output:

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

The calls to `thread::sleep` force a thread to stop its execution for a short duration, allowing a different thread to run. The threads will probably take turns, but that isn't guaranteed; it depends on how the operating system schedules the thread.

When main thread of a Rust program completes, all spawned threads are shut down, whether or not they have finished running. In the example output above, the main thread printed first, even though print statement from spawned thread appears first in the code. And even though we told the spawned thread to print until `i` is 9, it only got to 5 before the main thread shut down.

### Waiting for All Threads to Finish Using `join` Handles

To prevent spawned thread from being stopped prematurely, we can use `join` method. The return value of `thread::spawn` is `JoinHandle`, and when we call `join` method on it, it will wait for its thread to finish.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

Program output:

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Calling `join` on the handle blocks the thread currently running until the thread represented by the handle terminates.

If we move `handle.join()` before the `for` loop in `main, the order of thread execution will be different.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

Program output:

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

### Using `move` Closures with Threads

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

Compilation will fail:

```text
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0373]: closure may outlive the current function, but it borrows `v`, which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value `v`
7 |         println!("Here's a vector: {:?}", v);
  |                                           - `v` is borrowed here
  |
note: function requires argument type to outlive `'static`
 --> src/main.rs:6:18
  |
6 |       let handle = thread::spawn(|| {
  |  __________________^
7 | |         println!("Here's a vector: {:?}", v);
8 | |     });
  | |______^
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++

For more information about this error, try `rustc --explain E0373`.
error: could not compile `threads` due to previous error
```

The closure tries to borrow `v`, but Rust can't tell how long the spawned thread will run, so it does not know if the reference to `v` will always be valid.

Below is a scenario that's more likely to have a reference to `v` that won't be valid:

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
```

In code above, there's a possibility the spawned thread would be put in the background immediately without running at all. If main thread drops `v`, by the time spawned thread starts to execute, `v` is no longer valid.

To fix compiler error, follow error message's advice and add `move` keyword before the closure.

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

If we try to use the same fix on code above where main thread explicitly drops `v`, copmiler will fail for a different reason: `v` is moved into the closure's environment so it is no longer valid on the main thread.

Here's the compiler error:

```text
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0382]: use of moved value: `v`
  --> src/main.rs:10:10
   |
4  |     let v = vec![1, 2, 3];
   |         - move occurs because `v` has type `Vec<i32>`, which does not implement the `Copy` trait
5  |
6  |     let handle = thread::spawn(move || {
   |                                ------- value moved into closure here
7  |         println!("Here's a vector: {:?}", v);
   |                                           - variable moved due to use in closure
...
10 |     drop(v); // oh no!
   |          ^ value used here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `threads` due to previous error
```

## Using Message Passing to Transfer Data Between Threads

One increasingly popular approach to ensuring safe concurrency is _message passing_, where threads communicate by sending each other messages containing data. To accomplish message-sending concurrency, Rust's standard library provides an implementation of _channels_.

A channel has two halves: a transmitter and a receiver. A channel is said to be _closed_ if either the transmitter or receiver half is dropped.

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}
```

A new channel can be created using the `mpsc::channel` function; `mpsc` stands for _multiple producer, single consumer_. In short, the way Rust's standard library implements channels means a channel can have multiple _sending_ ends that produce values but only one _receiving_ end that consumes those values.

The `mpsc::channel` function returns a tuple of transmitter and the receiver.

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

Program output:

```text
Got: hi
```

The receiver has two useful methods: `recv` and `try_recv`.

The `recv`, short for _receive_, will block man thread's execution and wait until a value is sent down the channel. Once a value is sent, `recv` will return it in a `Result<T, E>`. When the transmitter closes, `recv` will return an error to signal that no more values will be coming.

The `try_recv` method does not block, but will instead return a `Result<T, E>` immediately: an `Ok` value holding a message if one is available and an `Err` value if there aren't any messages at this time. Using `try_recv` is useful if this thread has other work to do while waiting for messages.

### Channels and Ownership Transference

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

Compiler error:

```text
$ cargo run
   Compiling message-passing v0.1.0 (file:///projects/message-passing)
error[E0382]: borrow of moved value: `val`
  --> src/main.rs:10:31
   |
8  |         let val = String::from("hi");
   |             --- move occurs because `val` has type `String`, which does not implement the `Copy` trait
9  |         tx.send(val).unwrap();
   |                 --- value moved here
10 |         println!("val is {}", val);
   |                               ^^^ value borrowed here after move
   |
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0382`.
error: could not compile `message-passing` due to previous error
```

Here, we try to print `val` after it's sent down the channel, which could be modified or dropped in a different thread before it's being printed. The `send` function takes ownership of its parameter, and when value is moved, the receiver takes ownership of it. This stops us from accidentally using the value again after sending it.

### Sending Multiple Values and Seeing the Receiver Waiting

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

Program output:

```text
Got: hi
Got: from
Got: the
Got: thread
```

When running the code, the output should have a 1-second pause in between each line.

In the main thread, we are treating `rx` as an iterator. When channel is closed, iteration ends.

### Creating Multiple Producers by Cloning the Transmitter

```rust
    // --snip--

    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }

    // --snip--
```

Program output:

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

We may see values in different order in different runs. This is what makes concurrency interesting as well as difficult.

## Shared-State Concurrency

Another method of handling concurrency is to have multiple threads to access the same shared data. However, this often adds complexity. In a way channels are similar to single ownership, because once a value is transferred down a channel, that value is no longer valid in current thread. Shared memory concurrency is like multiple ownership as multiple threads can access the same data at the same time.

### Using Mutexes to Allow Access to Data from One Thread at a Time

_Mutex_ is an abbreviation for _mutual exclusion_, which allows only one thread to access some data at any given time. TO access data in a mutex, a thread must first signal that it wants access by asking to acquire the mutex's _lock_.

Mutexes have a reputation for being difficult to use because you have to remember two rules:

- You must attempt to acquire the lock before using the data.
- When you're done with the data that the mutex guards, you must unlock the data so other threads can acquire the lock.

#### The API of `Mutex`

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

As with many types, we create a `Mutex<T>` using the associated function `new`. To access the data inside the mutex, we use the `lock` method to acquire the lock, which will block the current thread.

The call to `lock` would fail if another thread holding the lock panicked. In that case, no one would ever be able to get the lock, so we can use `unwrap` and have this thread panic.

The type system ensures we acquire a lock before using the value in `m`, because `m` is of type `Mutex<i32>` and `lock` must be called to get the `i32` value.

The `Mutex<T>` is a smart pointer. More accurately, the call to `lock` returns a smart pointer called `MutexGuard`, wrapped in a `LockResult` that we handled with the call to `unwrap`. The `MutexGuard` smart pointer implements `Deref` to point to inner data, and the smart pointer also has a `Drop` implementation that releases the lock automatically when a `MutexGuard` goes out of scope, which happens at the end of inner scope in code example above.

#### Sharing a `Mutex` Between Multiple Threads

```rust
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Compilation error:

```text
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0382]: use of moved value: `counter`
  --> src/main.rs:9:36
   |
5  |     let counter = Mutex::new(0);
   |         ------- move occurs because `counter` has type `Mutex<i32>`, which does not implement the `Copy` trait
...
9  |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^ value moved into closure here, in previous iteration of loop
10 |             let mut num = counter.lock().unwrap();
   |                           ------- use occurs due to use in closure

For more information about this error, try `rustc --explain E0382`.
error: could not compile `shared-state` due to previous error
```

Compilation failed because we can't move the ownership of lock `counter` into multiple threads.

#### Multiple Ownership with Multiple Threads

The code below will attempt to wrap `Mutex<T>` in `Rc<T>` and clone the `Rc<T>` before moving ownership to the thread.

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Compilation error:

```text
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0277]: `Rc<Mutex<i32>>` cannot be sent between threads safely
  --> src/main.rs:11:36
   |
11 |           let handle = thread::spawn(move || {
   |                        ------------- ^------
   |                        |             |
   |  ______________________|_____________within this `[closure@src/main.rs:11:36: 11:43]`
   | |                      |
   | |                      required by a bound introduced by this call
12 | |             let mut num = counter.lock().unwrap();
13 | |
14 | |             *num += 1;
15 | |         });
   | |_________^ `Rc<Mutex<i32>>` cannot be sent between threads safely
   |
   = help: within `[closure@src/main.rs:11:36: 11:43]`, the trait `Send` is not implemented for `Rc<Mutex<i32>>`
note: required because it's used within this closure
  --> src/main.rs:11:36
   |
11 |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^
note: required by a bound in `spawn`
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:704:8
   |
   = note: required by this bound in `spawn`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `shared-state` due to previous error
```

Focus on error message `Rc<Mutex<i32>> cannot be sent between threads safely` and `Send is not implemented for Rc<Mutex<i32>>`. `Send` will be explained in the next section, it's one of the traits that ensures the types used with threads are meant for use in concurrent settings.

Unfortunately, `Rc<T>` is not safe to share across threads. When `Rc<T>` manages the reference count, it adds to the count for each call to `clone` and subtracts from the count when each clone is dropped, but it does not use any concurrency primitives to make sure that changes to the count can't be interrupted by another thread, which could lead to wrong counts which could in turn lead to memory leaks or a value being dropped before we are done with it.

#### Atomic Reference Counting with `Arc`

Fortunately, `Arc<T>` is a type like `Rc<T>` that is safe to use in concurrent situations. `Arc` stands for _atomically reference counted_ type. Atomics are an additional kind of concurrency primitive that won't be covered in detail here; see standard library documentation for `std::sync::atomic` for more details.

The reason why primitive types aren't atomic is because thread safety comes with a performance penalty. If our code runs on a single thread, it does not need to have atomicity guarantees.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Program output:

```text
Result: 10
```

Note that for simple numerical operations, there are types simpler than `Mutex<T>` types provided by `std::sync::atomic` module of the standard library. Those types provide safe, concurrent, atomic access to primitive types.

### Similarities Between `RefCell`/`Rc` and `Mutex`/`Arc`

`Mutex<T>` provides interior mutability, as the `Cell` family does. In the same way we use `RefCell<T>` to mutate contents inside an `Rc<T>`, we use `Mutex<T>` to mutate contents inside an `Arc<T>`.

Another note, Rust can't protect us from all kinds of logic errors when using `Mutex<T>`. Using `Rc<T>` comes with the risk of creating reference cycles, causing memory leaks. Similarly, `Mutex<T>` comes with risk of creating deadlocks, which occurs when an operation needs to lock two resources and two threads have each acquired one of the locks, causing both to wait for each other forever.

## Extensible Concurrency with the `Sync` and `Send` Traits

Rust language has very few concurrency features. Almost every concurrency feature in this chapter has been part of the standard library, not the language.

However, two concurrency concepts are embedded in the language: the `std::marker` traits `Sync` and `Send`.

### Allowing Transference of Ownership Between Threads with `Send`

The `Send` marker trait indicates that ownership values of the type implementing `Send` can be transferred between threads. Almost every Rust type is `Send`, with a few exceptions, including `Rc<T>`, which cannot be `Send` because its reference count may be updated at the same time by different threads.

Any type composed entirely of `Send` types is automatically marked as `Send` as well. Almost all primitive types are `Send`, aside from raw pointers which will be discussed in Chapter 19.

### Allowing Access from Multiple Threads with `Sync`

The `Sync` marker trait indicates that it is safe for the type implementing `Sync` to be referenced from multiple threads. In other words, any type `T` is `Sync` if `&T` (immutable reference to `T`) is `Send`, meaning reference can be sent safely to another thread. Similar to `Send`, primitive types are `Sync`, and types composed entirely of types that are `Sync` are also `Sync`.

Smart pointer `Rc<T>` is also not `Sync` for the same reasons that it's not `Send`. The `RefCell<T>` type and the family of related `Cell<T>` types are not `Sync`. The implementation of borrow checking that `RefCell<T>` does at runtime is not thread-safe. The smart pointer `Mutex<T>` is `Sync` and can be used to share access with multiple threads.

### Implmmenting `Send` and `Sync` Manually is Unsafe

Because types that are made up of `Send` and `Sync` traits are automatically also `Send` and `Sync`, we don't have to implement those traits manually. As marker traits, they don't even have any methods to implement; they are just useful for enforcing invariants related to concurrecy.

Manually implementing these traits involves implementing unsafe Rust code, which will be discussed in Chapter 19. For now, it's important to know that building new concurrent types not made up of `Send` and `Sync` traits requires careful thought to uphold the safety guarantees.
