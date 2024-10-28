<!-- markdownlint-disable MD013 -->

# Structs

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Structs](#structs)
  - [Defining and Instantiating Structs](#defining-and-instantiating-structs)
    - [Struct Update Syntax](#struct-update-syntax)
    - [Tuple Structs](#tuple-structs)
    - [Unit-Like Structs](#unit-like-structs)
    - [Ownership of struct data](#ownership-of-struct-data)
  - [An example program using structs](#an-example-program-using-structs)
  - [Method Syntax](#method-syntax)
    - [Defining Methods](#defining-methods)
    - [Automatic referencing and dereferencing](#automatic-referencing-and-dereferencing)
    - [Associated Functions](#associated-functions)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Defining and Instantiating Structs

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

To modify a field value in struct, the entire instance must be marked mutable.

```rust
let mut user2 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
user1.email = String::from("anotheremail@example.com");
```

Function to build user instances:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

Use field init shorthand in `build_user` function:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

### Struct Update Syntax

```rust
// Equivalent
let user2 = User {
    active: user1.active,
    username: user1.username,
    email: String::from("another@example.com"),
    sign_in_count: user1.sign_in_count,
};
let user2 = User {
    email: String::from("another@example.com"),
    ..user1
};
```

Note we can no longer use user1 since `username` field of user1 is moved to user2.

### Tuple Structs

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

We can use `.` followed by index to access an individual value in a struct instance.

### Unit-Like Structs

```rust
struct AlwaysEqual; // unit-like structs without any fields, similar to `()` tuple.
let subject = AlwaysEqual;
```

### Ownership of struct data

It is possible for structs to store references to data owned by something else, but to do so requires use of _lifetime_, a Rust feature discussed in chapter 10.

## An example program using structs

To print structs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2
    let rect1 = Rectangle {
        width: dbg!(30 * scale), // dbg! prints to stderr, returns ownership of expression's value
        height: 50,
    };
    println!("rect1 is {:?}", rect1);
    println!("rect1 is {:#?}", rect1); // pretty-print with {:#?}
    dbg!(&rect1); // dbg! prints to stderr, use &rect1 so dbg! does not take ownership
}
```

## Method Syntax

### Defining Methods

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
    // Method can have same name as one of the struct's field
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

### Automatic referencing and dereferencing

When you call a method called `object.something()`, Rust automatically adds in `&`, `&mut` or `*` so `object` matches signature of the method.

```rust
// Equivalent
p1.distance(&p2);
(&p1).distance(&p2);
```

### Associated Functions

All functions defined in `impl` block are called associated functions.
We can define associated functions that don't have `self` as their first parameter

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}

fn main() {
    let sq = Rectangle::square(3);
}
```

Each struct is allowed to have multiple `impl` blocks.
