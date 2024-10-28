<!-- markdownlint-disable MD013 -->

# Ownership

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Ownership](#ownership)
  - [Ownership Basics](#ownership-basics)
    - [Stack and Heap](#stack-and-heap)
      - [Ownership Rules](#ownership-rules)
    - [Variable Scope](#variable-scope)
    - [The `String` Type](#the-string-type)
    - [Memory And Allocation](#memory-and-allocation)
      - [Variable-Data Interaction: Move](#variable-data-interaction-move)
      - [Variable-Data Interaction: Clone](#variable-data-interaction-clone)
      - [Stack-Only Data copy](#stack-only-data-copy)
    - [Ownership and Functions](#ownership-and-functions)
    - [Return Values and Scope](#return-values-and-scope)
  - [References and Borrowing](#references-and-borrowing)
    - [Immutable References](#immutable-references)
    - [Mutable References](#mutable-references)
      - [Dangling Pointers](#dangling-pointers)
      - [The rules of references](#the-rules-of-references)
  - [The Slice Type](#the-slice-type)
    - [String Slices](#string-slices)
    - [String Literals Are Slices](#string-literals-are-slices)
    - [String Slices as Parameters](#string-slices-as-parameters)
    - [Other Slices](#other-slices)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Ownership Basics

### Stack and Heap

- Stack is FILO. Heap is less structured, when we put data on the heap, memory allocator finds an empty spot on heap big enough to hold data, mark it as used and returns a pointer.
- Pushing to stack is faster than allocating on the heap because allocator never has to search for new place to store data.
- Accessing data on heap is slower than stack because you have to follow a pointer to get data.
- When calling a function, values passed into the function and function's local variable gets pushed onto the stack, and they are popped off the stack when function is over.

#### Ownership Rules

- Each value in Rust has a variable that’s called its owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

### Variable Scope

```rust
{                      // s is not valid here, it’s not yet declared
    let s = "hello";   // s is valid from this point forward
    // do stuff with s
}                      // this scope is now over, and s is no longer valid
```

### The `String` Type

String literals must be defined at compile time and are immutable. Use `String` type for mutable strings.

```rust
let mut s = String::from("hello");
s.push_str(", world!"); // `hello, world!`
```

### Memory And Allocation

When a variable goes out of scope, Rust calls special function `drop` that deallocates memory used by variable.

#### Variable-Data Interaction: Move

When a variable is copied, the value is moved rather than copied. This rule prevents double free when both variables go out of scope.

```rust
let s1 = String::from("hello");
let s2 = s1; // s1 value moved to s2, s2 now holds "hello"
println!("{}, world!", s1); // Compile error: value borrowed here after move
```

#### Variable-Data Interaction: Clone

Clone for deep copy

```rust
let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2); // s1 = hello, s2 = hello
```

#### Stack-Only Data copy

Types with known size at compiled time are stored entirely on stack, so copies of values are quick to make.

```rust
let x = 5;
let y = x;
println!("x = {}, y = {}", x, y); // x = 5, y = 5
```

Rust has a special annotation called the `Copy` trait that can be placed on types stored on stack.
If a type implements `Copy` trait, a variable is still valid after assignment to another variable.
Rust won't let us annotate a type with `Copy` trait if the type or any of its parts has implemented `Drop` trait.

Any group of scalar groups, types that does not require allocation can implement `Copy`:

- All integer types
- Boolean
- All floating-point types
- Character type
- Tuples, if they only contain types that also implements `Copy`

### Ownership and Functions

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope
    takes_ownership(s); // s's value moves into the function...
                        // ... and so is no longer valid here
    let x = 5;  // x comes into scope
    makes_copy(x);  // x would move into the function,
                    // but i32 is Copy, so it's okay to still
                    // use x afterward
} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

### Return Values and Scope

```rust
fn main() {
    let s1 = gives_ownership(); // gives_ownership moves its return value into s1
    let s2 = String::from("hello"); // s2 comes into scope
    let s3 = takes_and_gives_back(s2);  // s2 is moved into takes_and_gives_back,
                                        // which also moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {    // gives_ownership will move its return value
                                    // into the function that calls it
    let some_string = String::from("yours"); // some_string comes into scope
    some_string // some_string is returned and moves out to the calling function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into scope
    a_string  // a_string is returned and moves out to the calling function
}
```

Ownership makes returning values from a function a bit tedious. We could return multiple values using a tuple, but that's also tedious. Luckily, references make this easier.

## References and Borrowing

### Immutable References

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len); // The length of hello is 5.
}

fn calculate_length(s: &String) -> usize {
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, nothing happens.
```

We call the action of creating a reference borrowing. We cannot modify borrowed references.

```rust
fn main() {
    let s = String::from("hello");
    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world"); // Compile error: `some_string` is a `&` reference,
                                     // so the data it refers to cannot be borrowed as mutable
}
```

### Mutable References

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

Mutable references have one big restriction: you can have only one mutable reference to a particular piece of data at a time.
This code that attempts to create two mutable references to s will fail:

```rust
let mut s = String::from("hello");
let r1 = &mut s; // First mutable borrow
let r2 = &mut s; // Second mutable borrow
println!("{}, {}", r1, r2); // First borrow used. Compilation fails.
```

This helps prevent data races at compile time. A data race happens when these 3 behaviors occurs:

- Two or more pointers access the same data at the same time.
- At least one of the pointers is being used to write to the data.
- There’s no mechanism being used to synchronize access to the data.

We can use curly brackets to create a new scope, allowing multiple mutable references, just not _simultaneous_ ones:

```rust
let mut s = String::from("hello");
{
    let r1 = &mut s;
} // r1 goes out of scope here, so we can make a new reference with no problems.
let r2 = &mut s;
```

Rust enforces rule for combining mutable immuatable references, we cannot have a mutable reference while we have an immutable one to the same value.

```rust
let mut s = String::from("hello");
let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM
println!("{}, {}, and {}", r1, r2, r3); // Compile error: immutable borrow later used here
```

```rust
let mut s = String::from("hello");
let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// variables r1 and r2 will not be used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```

#### Dangling Pointers

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle returns a reference to a String
    let s = String::from("hello");
    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Compile error! Solution is to return String directly

fn no_dangle() -> String {
    let s = String::from("hello");
    s // Ownership is moved out, and nothing is deallocated.
}
```

#### The rules of references

- At any given time, you can have _either_ one mutable reference or any number of immutable references.
- References must always be valid.

## The Slice Type

_Slices_ let you reference a contiguous sequence of elements in a collection rather than whole collection. A slice is a kind of reference, so it does not have ownership

Example:

```rust
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s); // word will get the value 5
    s.clear(); // this empties the String, making it equal to ""

    // word still has the value 5 here, but there's no more string that
    // we could meaningfully use the value 5 with.
    // This code would compile, but if we attempt to access `s[word]`,
    // compilation error!
}

// Returns the length of first word
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

### String Slices

```rust
let s = String::from("hello world");
let hello = &s[0..5];
let world = &s[6..11];

// Equivalent
let slice1 = &s[0..2];
let slice2 = &s[..2];

// Equivalent
let slice3 = &s[3..len];
let slice4 = &s[3..];

// Equivalent
let slice5 = &s[0..len];
let slice6 = &s[..];
```

Rewrite `first_word` to return a slice:

```rust
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s); // Immutable borrow
    s.clear(); // Mutable borrow
    println!("the first word is: {}", word); // Immutable borrow, compile error.
}

fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

### String Literals Are Slices

```rust
let s = "Hello, world!"; // s is of type &str, a slice pointing a specific point
                         // of the binary, which is why string literals are immutable;
                         // &str is an immuatable reference.
```

### String Slices as Parameters

Knowing `first_word` can take slices of literals and `String` values lead to improvement to its signature:

```rust
fn first_word(s: &String) -> &str {
// to
fn first_word(s: &str) -> &str {
```

If we have a string slice, we can pass that directly. If we have a `String`, we can pass a slice of the `String` or a reference to the `String`.
This flexibility takes advantage of `deref coercions`, a feature covered in later chapter.

```rust
fn main() {
    let my_string = String::from("hello world");

    // `first_word` works on slices of `String`s, whether partial or whole
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // `first_word` also works on references to `String`s, which are equivalent
    // to whole slices of `String`s
    let word = first_word(&my_string);

    let my_string_literal = "hello world";

    // `first_word` works on slices of string literals, whether partial or whole
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

### Other Slices

```rust
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3];
assert_eq!(slice, &[2, 3]);
```
