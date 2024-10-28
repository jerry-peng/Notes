<!-- markdownlint-disable MD013 -->

# Object-Oriented Programming Features of Rust

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Object-Oriented Programming Features of Rust](#object-oriented-programming-features-of-rust)
  - [Chareteristics of Object-Oriented Languages](#chareteristics-of-object-oriented-languages)
  - [Using Trait Objects That Allow for Values of Different Types](#using-trait-objects-that-allow-for-values-of-different-types)
    - [Defining a Trait for Common Behavior](#defining-a-trait-for-common-behavior)
    - [Implementing the Trait](#implementing-the-trait)
    - [Trait Objects Perform Dynamic Dispatch](#trait-objects-perform-dynamic-dispatch)
  - [Implementing an Object-Oriented Design Pattern](#implementing-an-object-oriented-design-pattern)
    - [Defining `Post` and Creating a New Instance in the Draft State](#defining-post-and-creating-a-new-instance-in-the-draft-state)
    - [Storing the Text of the Post Content](#storing-the-text-of-the-post-content)
    - [Ensuring the Content of a Draft Post Is Empty](#ensuring-the-content-of-a-draft-post-is-empty)
    - [Requesting a Review of the Post Changes Its State](#requesting-a-review-of-the-post-changes-its-state)
    - [Adding `approve` to Change the Behavior of `content`](#adding-approve-to-change-the-behavior-of-content)
    - [Trade-offs of the State Pattern](#trade-offs-of-the-state-pattern)
      - [Encoding States and Behaviors as Types](#encoding-states-and-behaviors-as-types)
      - [Implementing Transitions as Transformations into Different Types](#implementing-transitions-as-transformations-into-different-types)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Chareteristics of Object-Oriented Languages

- Objects contain data and behavior
- Encapsulation that hides implementation details
- Inheritance and polymorphism

## Using Trait Objects That Allow for Values of Different Types

We'll attempt to implement a GUI library to demonstrate inheritance using Rust's traits. For the example project, we'll define a class named `Component` with method named `draw`, and we'll create sub classes such as `Button`, `Image`, `SelectBox`.

### Defining a Trait for Common Behavior

A _trait object_ points to both an instance of type implementing our specified trait and a table used to look up trait methods on that type at runtime. Trait object can be created by specifying some sort of pointer, such as a `&` reference or a `Box<T>` smart pointer, then the `dyn` keyword, and then specifying the relevant trait. We'll discuss the reason trait objects must use a pointer in Chapter 19.

Whenever we use a trait objects, Rust's type system will ensure a compile time that any value used in that context will implement the trait object's trait. Consequently, we don't need to know all the possible types at compile time.

```rust
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

The `components` vector is of type `Box<dyn Draw>`, which is a trait object; it's a stand-in for any type inside a `Box` that implements the `Draw` trait.

This works differently from defining a struct that uses a generic type parameter with trait bounds. A generic type parameter can only be substituted with one concrete type at a time, whereas trait objects allow for multiple concrete types to fill in for the trait object at runtime.

Below is an example of generic type parameter with trait bounds:

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
where
    T: Draw,
{
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

This restricts us to a `Screen` instance that has a list of homogeneous components all of type `Button` or all of type `TextField`. If we only have homogeneous collections, using generics and trait bounds is preferable because definitions will be monomorphized at compile time.

### Implementing the Trait

```rust
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // code to actually draw a button
    }
}
```

```rust
use gui::Draw;

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}
```

```rust
use gui::{Button, Screen};

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

The advantage of using trait objects is that we never have to check whether a value implements a particular method at runtime or worry about getting errors if a value does not implement a method but we call it anyway. Rust won't compile our code if the values don't implement the traits that the trait objects need.

For example:

```rust
use gui::Screen;

fn main() {
    let screen = Screen {
        components: vec![Box::new(String::from("Hi"))],
    };

    screen.run();
}
```

Compilation error:

```text
$ cargo run
   Compiling gui v0.1.0 (file:///projects/gui)
error[E0277]: the trait bound `String: Draw` is not satisfied
 --> src/main.rs:5:26
  |
5 |         components: vec![Box::new(String::from("Hi"))],
  |                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `Draw` is not implemented for `String`
  |
  = help: the trait `Draw` is implemented for `Button`
  = note: required for the cast from `String` to the object type `dyn Draw`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `gui` due to previous error
```

### Trait Objects Perform Dynamic Dispatch

Recall that code using generics goes through monomorphization process performed by the compiler to generate nongeneric implementations of functions and methods for each concrete type in place of generic type parameter. The code that results from monomorphization is doing _static dispatch_, which is when compiler knows what method we are calling at compile time. This is in contrast to _dynamic dispatch_, which is when the compiler can't tell at compile time which method we are calling. In dynamic dispatch cases, the compiler emits code that will figure out which method to call at runtime.

When using trait objects, Rust must use dynamic dispatch, because the compiler does not know all the types that might be used with the code that's using trait objects, so it does not know which method implemented on which type to call. Instead, at runtime, Rust uses the pointers inside the trait object to know which method to call. This lookup incurs a runtime cost, and it also prevents the compiler from choosing to inline a method's code which prevents some optimizations. Trait objects gives extra flexibility at some performance cost, so it's a trade-off.

## Implementing an Object-Oriented Design Pattern

We'll implement a blog post workflow incrementally using the state pattern.

Final functionality:

1. A blog post starts as an empty draft
1. When draft is done, a review of the post is requested
1. When the post is approved, it gets published.
1. Only published blog posts return content to print, so unapproved posts can't be published accidentally.

Here's an example usage of the API we'll implement in a library create named `blog`, which of course won't compile just yet.

```rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

### Defining `Post` and Creating a New Instance in the Draft State

```rust
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
}

trait State {}

struct Draft {}

impl State for Draft {}
```

The `state` field is set to new instance of the `Draft` struct, which ensures whenever we create a new instance of `Post`, it will start out as a draft. Because `state` field is private, there is no way to create a `Post` in any other state.

### Storing the Text of the Post Content

```rust
impl Post {
    // --snip--
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

Instead of exposting `content` field as `pub`, we implement `add_text` method, so that later we can implement a method that will control how `content` field's data is read.

### Ensuring the Content of a Draft Post Is Empty

```rust
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        ""
    }
}
```

### Requesting a Review of the Post Changes Its State

```rust
impl Post {
    // --snip--
    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

The `request_review` method is added to the `State` trait; all types that implement the trait will now need to implement the method. Rather than having `self`, `&self`, or `&mut self` as the first parameter of the method, we have `self: Box<Self>`, which means the method is only valid when called on a `Box` holding the type. This syntax takes ownership of `Box<Se.f`, invalidating the old state so the state value of the `Post` can transform into a new state.

To consume the old state, and `take` method take the `Some` value out of the `state` field and leave a `None` in its place.

We can start seeing the advantages of the state pattern: the `request_review` method on `Post` is the same no matter its `state` value. Each state is responsible for its own rules.

### Adding `approve` to Change the Behavior of `content`

```rust
impl Post {
    // --snip--
    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    // --snip--
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    // --snip--
    fn approve(self: Box<Self>) -> Box<dyn State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
```

The code above adds `approve` method to the `State` trait, which only takes effect on `PendingReview` struct.

To return value from `content` depending on current state of `Post`, we can delegate it to structs that implement `State`.

```rust
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(self)
    }
    // --snip--
}
```

We call the `as_ref` method on the `Option` because we want a reference to the value inside the `Option` rather than ownership. Because `state` is `Option<Box<dyn State>>`, when we can `as_ref`, `Option<&Box<dyn State>>` is returned. If `as_ref` is not called, compilation will fail as we cannot move `state` out of the borrowed `&self` function parameter.

The `unwrap` method is used because we know it will never panic because the methods on `Post` ensure the `state` will always contain a `Some` value, even though compiler can't understand it.

```rust
trait State {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

// --snip--
struct Published {}

impl State for Published {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```

We add a default implementation for the `content` method that returns an empty string slice, so we don't need to implement `content` on the `Draft` and `PendingReview` structs. Also note that lifetime annotations are needed on this method because lifetime of the returned references is related to the lifetime of the `post` argument.

### Trade-offs of the State Pattern

With object-oriented state pattern, methods on `Post` know nothing about the various behaviors, and we only need to look in one place to know the different ways a published post can behave: the implementation of the `State` trait on the `Published` struct.

If we were to create an alternative implementation that did not use state pattern, we might instead use `match` expressions, which would mean we would have to look in several places to understand, and `match` expression would need more arms as more states are added.

One downside of the state pattern is some of the states are coupled to each other. If we add another state between `PendingReview` and `Published`, such as `Scheduled`,we would have to change the code in `PendingReview` to transition to `Scheduled` instead. It would be less work if `PendingReview` did not need to change with addition of new state, but that would mean switching to another design pattern.

Another downside is we have some duplicate logic. Some may try to make default implementations for `request_review` and `approve` methods on `State` trait that return `self`; that won't work, because it would violate object safety, because the trait does not know what the concrete `self` will be exactly. If we want to use `State` as a trait object, its methods need to be object safe.

Other duplication includes similar implementations of the `request_review` and `approve` methods on `Post`. Both methods delegate to the implementation of the same method on the value in the `state` field of `Option` and set the new value of the `state` field to the result. We may consider defining a macro to eliminate the repetition, which will be discussed in Chapter 19.

We can make some changes that can make invalid states and transitions into compile time errors.

#### Encoding States and Behaviors as Types

Another set of trade-offs; rather than encapsulating the states and transitions completely, we'll encode the states into different types. Rust's type checking system will prevent attempts to use draft posts where only published posts are allowed by issuing a compile error.

```rust
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
        &self.content
    }
}

impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());
}
```

The draft posts don't have the `content` method at all, so if any code attempts to get a draft post's content, compilation will fail.

#### Implementing Transitions as Transformations into Different Types

```rust
impl DraftPost {
    // --snip--
    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

pub struct PendingReviewPost {
    content: String,
}

impl PendingReviewPost {
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}
```

The `request_review` and `approve` methods take ownership of `self`, thus consuming the `DraftPost` and `PendingReviewPost` instances. We have now encoded the blog post workflow into the type system.

Since `request_review` and `approve` methods return new instances rather than modifying the struct they're called on, we need to add more `let post =` shadowing assignment to save the returned instances. We also can't have assertions about draft and pending review posts' contents be empty strings, because it would cause compilation error.

```rust
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");

    let post = post.request_review();

    let post = post.approve();

    assert_eq!("I ate a salad for lunch today", post.content());
}
```

This pattern does not quite follow the object-oriented state pattern anymore. However, the gain is that invalid states are now impossible because of the type system and the type checking that happens at compile time, such as display of content of an unpublished post.
