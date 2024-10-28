<!-- markdownlint-disable MD013 -->

# Common Collections

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Common Collections](#common-collections)
  - [Vector](#vector)
    - [Creating a New Vector](#creating-a-new-vector)
    - [Updating a Vector](#updating-a-vector)
    - [Dropping a vector drop its element](#dropping-a-vector-drop-its-element)
    - [Reading Elements of Vectors](#reading-elements-of-vectors)
    - [Iterating Over the Values in a Vector](#iterating-over-the-values-in-a-vector)
    - [Using an Enum to Store Multiple Types](#using-an-enum-to-store-multiple-types)
  - [Storing UTF-8 Encoded Text with Strings](#storing-utf-8-encoded-text-with-strings)
    - [Creating a New String](#creating-a-new-string)
    - [Updating a String](#updating-a-string)
      - [Appending to a string](#appending-to-a-string)
      - [String Concatenation](#string-concatenation)
    - [Indexing Into Strings](#indexing-into-strings)
      - [Internal Representation](#internal-representation)
      - [Bytes and Scalar Values and Graphene Clusters](#bytes-and-scalar-values-and-graphene-clusters)
    - [Slicing Strings](#slicing-strings)
  - [Methods for Iterating Over Strings](#methods-for-iterating-over-strings)
  - [Hash Maps](#hash-maps)
    - [Creating a New Hash Map](#creating-a-new-hash-map)
    - [Hash Maps and Ownership](#hash-maps-and-ownership)
    - [Accessing Values in Hash Map](#accessing-values-in-hash-map)
    - [Updating a Hash Map](#updating-a-hash-map)
      - [Overwriting a value](#overwriting-a-value)
      - [Only Inserting a Value If the Key Has No Value](#only-inserting-a-value-if-the-key-has-no-value)
      - [Updating a Value Based on Old Value](#updating-a-value-based-on-old-value)
    - [Hashing Function](#hashing-function)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Vector

### Creating a New Vector

```rust
let v: Vec<i32> = Vec::new();
```

In most code, Rust can often infer the type of value stored in vector, so type annotation is rarely needed.

It is more common to create a `Vec<T>` that has initial values, Rust provides the `vec!` macro for convenience.

```rust
let v = vec![1, 2, 3];
```

### Updating a Vector

```rust
let mut v = Vec::new();
v.push(5);
```

### Dropping a vector drop its element

```rust
{
    let v = vec![1, 2, 3, 4];

    // do stuff with v
} // <- v goes out of scope and is freed here
```

### Reading Elements of Vectors

Elements can be accessed through indexing syntax (`&` and `[]`), which returns a reference, or `get` method, which returns `Option<&T>`.

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {}", third);

match v.get(2) {
    Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
}
```

When accessing index over bound:

```rust
let v = vec![1, 2, 3, 4, 5];
let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

The `[]` method will cause program to panic, the `get` would return `None` without panicking.

The borrow checker enforces ownership and borrowing rules to ensure to the contents of vector remain valid.

```rust
let mut v = vec![1, 2, 3, 4, 5];
let first = &v[0]; // Immutable borrow
v.push(6); // Mutable borrow
println!("The first element is: {}", first); // Compile error: immutable borrow later used here
```

The code above looks like it should work, why should a reference to the first element care about what changes at the end of the vector?

This is due to the way vectors work, adding a new element to the end of vector might require re-allocating larger memory block and copying over old vector items onto the new memory block.
In which case, the reference to first element would be pointing to deallocated memory.

### Iterating Over the Values in a Vector

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

Iterate over mutable references to each element in mutable vector to make changes to all elements.

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

To change value the mutable reference refers to, we have to use dereference operator (`*`) to get the value in `i` before we can use `+=` operator.

### Using an Enum to Store Multiple Types

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

If we don't know exhaustive set of types the program will get at runtime to store in a vector, enum technique won't work. Instead, we can use trait object, which will be discussed in later chapters.

## Storing UTF-8 Encoded Text with Strings

Rust has only one string type in the core language, which is the string slice `str` usually seen in its borrowed form `&str`. In Chapter 4, we talked about string slices, which are references
to some UTF-8 encoded string data stored elsewhere. String literals, are stored in program's binary and are therefore string slices.

The `String` type, is provided by Rust's standard library rather than in core language, which is a growable, mutable, owned, UTF-8 encoded string type.

Rust's standard library also includes a number of other string types, such as `OsString`, `OsStr`, `CString` and `CStr`. The `String` and `Str` refer to owned and borrowed variants.
These strings can store text in different encodings or be represented in memory in a different way.

### Creating a New String

To create a new empty string.

```rust
let mut s = String::new();
```

If we have initial data to start the string with, we use the `to_string` method, which is available on any type that implements the `Display` trait, as string literals do.

```rust
let data = "initial contents";
let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();
```

We can also use `String::from` to create `String` from string literal.

```rust
let s = String::from("initial contents");
```

Strings are UTF-8 encoded, so we can include any properly encoded data in them.

```rust
let hello = String::from("السلام عليكم");
let hello = String::from("Dobrý den");
let hello = String::from("Hello");
let hello = String::from("שָׁלוֹם");
let hello = String::from("नमस्ते");
let hello = String::from("こんにちは");
let hello = String::from("안녕하세요");
let hello = String::from("你好");
let hello = String::from("Olá");
let hello = String::from("Здравствуйте");
let hello = String::from("Hola");
```

### Updating a String

#### Appending to a string

The `push_str` method takes a string slice instead of taking ownership of the parameter.

```rust
let mut s = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {}", s2); // "s2 is foobar"
```

The `push` method pushes a single character to the `String`.

```rust
let mut s = String::from("lo");
s.push('l');
```

#### String Concatenation

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

The reason `s1` is no longer valid after the addition, and the reason we used a reference to `s2` has to do with the signature of the method called when we use `+` operator.
`+` operator uses the `add` method, something like:

```rust
fn add(self, s: &str) -> String {
    // --snip--
}
```

The exact signature for `add` is defined using generics in standard library. But, looking at the signature above gives us clues to understand tricky bits of `+` operator.

First, because of `s` parameter in `add` function, we can only add a `&str` to a `String`; we can't add two `String` values together.
However, `&s2` is of type `&String` not `&str`, why is it valid? The reason we're able to use `&s2` in the call to `add` is the compiler can _coerce_ the `&String` argument into a `&str`.
When calling `add` method, Rust uses a _deref coercion_, which turns `&s2` into `&s2[..]`, which will be discussed in Chapter 15.

Second, `add` takes ownership of `self`, because `self` does not have an `&`. This means `s1` in example above will be moved into `add` call and no longer be valid.
So although `let s3 = s1 + &s2;` looks like it will copy both strings and create a new one, the statement actually takes ownership of `s1`, appends a copy of contents of `s2`,
and returns ownership of the result. This is more efficient than copying.

If we need to concatenate multiple strings, we can use `+` which can be hard to read, or we can use `format!` macro.

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
// is equivalent to
let s = format!("{}-{}-{}", s1, s2, s3);
```

### Indexing Into Strings

```rust
let s1 = String::from("hello");
let h = s1[0];
// Compile error: `String` cannot be indexed by `{integer}`
```

Rust does not support accessing part of a string using indexing syntax, which will be explained below.

#### Internal Representation

A `String` is a wrapper over a `Vec<u8>`.

```rust
let hello = String::from("Hola");
```

In case above, `len` will be 4, which means the vector storing string "Hola" is 4 bytes long.

```rust
let hello = String::from("Здравствуйте");
```

The string above has 12 characters but has a `len` of 24, which is the number of bytes it takes to encode the string in UTF-8, because each Unicode scalar value in the string takes 2 bytes.
Therefore, index into string's bytes will not correlate to a valid Unicode scalar value. Consider this invalid Rust code:

```rust
let hello = "Здравствуйте";
let answer = &hello[0];
```

What should `answer` be? `З`? When encoded in UTF-8, the first byte of 3 is 208 and second is 151, so `answer` should be 208, but 208 does not represent a valid character on its own.
Users generally don't want byte value returned. To avoid returning an unexpected value and causing bugs not discovered immediately, Rust does not compile this code to prevent misunderstandings.

#### Bytes and Scalar Values and Graphene Clusters

If we look at Hindi word "नमस्ते", it is stored as a vector of `u8`:

```rust
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164, 224, 165, 135]
```

That's 18 bytes, if we look at them as Unicode scalar values, which are Rust's `char` type, the bytes look like:

```rust
['न', 'म', 'स', '्', 'त', 'े']
```

There are six `char` values, but the forth and sixth are not letters, they're diacritics that don't make sense on their own. Finally if we look at them as graphene clusters,
we'd get four letters that make up the Hindi word:

```rust
["न", "म", "स्", "ते"]
```

A final reason Rust doesn't allow us to index into `String` to get a character is that, indexing operations are expected to take constant time (O(1)). But it isn't possible to guarantee that performance with a string.

### Slicing Strings

Indexing into string is often a bad idea due to ambiguity of the return type. Therefore, to create a string slice, we can use `[]` with a range to create string slice from particular bytes:

```rust
let hello = "Здравствуйте";
let s = &hello[0..4];
```

`s` will be `&str` that contains first 4 bytes of the string, which means `s` is `Зд` because these characters are 2 bytes.

If we use `&hello[0..1]`, Rust would panic at runtime the same way as if an invalid index were accessed in a vector.

## Methods for Iterating Over Strings

If need to perform operations on individual Unicode scalar values, best way to do so is to use `chars` method.

```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

prints:

```rust
न
म
स
्
त
े
```

`bytes` method returns each raw byte:

```rust
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

prints:

```rust
224
164
// --snip--
165
135
```

Remember valid Unicode scalar values may be made up of more than 1 byte.

## Hash Maps

### Creating a New Hash Map

Of the three common collections, hash map is the least often used, so it's not included in the prelude, and needs to be imported from standard library

```rust
use std::collections::HashMap;
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

Like vector, hash maps are homogeneous, all keys must have the same type, so are all values.

Another way to create hash map is by using iterators and `collect` method on a vector of tuples, where each tuple consists of a key and its value.

```rust
use std::collections::HashMap;

let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let mut scores: HashMap<_, _> = teams.into_iter().zip(initial_scores.into_iter()).collect();
```

`zip` method creates iterator of tuples where "Blue" is paired with 10, and so on.

Type annotation `HashMap<_, _>` is needed because `collect` can return many different data structures and Rust doesn't know which is wanted unless specified.
The underscores for key value type is because Rust can infer types that hash map contains based on types of data in vectors.

### Hash Maps and Ownership

For types that implement `Copy` trait, like `i32`, values are copied into hash map. Other types are moved into hash map.

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point because they are moved into map
```

If we insert references to values into hash map, values won't be moved into the hash map. The values that references point to must be valid for at least as long as the hash map is valid.

### Accessing Values in Hash Map

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

`score` is `Some(&10)`. It is `Some` because `get` returns an `Option<&V>`, if there is no value for a specific key in hash map, `get` will return `None`.

To iterate key/value pairs in hash maps, use a `for` loop.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

which prints:

```rust
Yellow: 50
Blue: 10
```

### Updating a Hash Map

#### Overwriting a value

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores); // {"Blue":25}, 10 has been overwritten
```

#### Only Inserting a Value If the Key Has No Value

Hash map `entry` method checks whether a key has a value, and if it doesn't, insert the value for it. The return value is an enum called `Entry`.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores); // {"Yellow": 50, "Blue": 10}
```

The `or_insert` method on `Entry` returns a mutable reference to the value for the corresponding `Entry` key if that key exists, otherwise inserts the parameter as new value for the key and returns mutable reference to the new value.

#### Updating a Value Based on Old Value

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map); // {"world": 2, "hello": 1, "wonderful": 1}
```

The `split_whitespace` method iterates over sub-slices, separated by whitespace. The `or_insert` method returns a mutable reference to the value of key. In order to assign to
that value, we must dereference it using asterisk (`*`). The mutable reference goes out of scope at the end of `for` loop, so all changes are safe and allowed by borrowing rules.

### Hashing Function

By default, `HashMap` uses `SipHash` that can provide resistance to Denial of Service (DoS) attacks involving hash tables. This is not the fastest algorithm available, but trade-off for better
security is worth it. If default hash function is too slow, we can switch to another function by specifying a different _hasher_. A hasher is a type that implements the `BuildHasher` trait.
