<!-- markdownlint-disable MD013 -->

# Common Programming Concepts

<!-- prettier-ignore-start -->

<!--toc:start-->
- [Common Programming Concepts](#common-programming-concepts)
  - [Variables and Mutability](#variables-and-mutability)
    - [immutable variables](#immutable-variables)
    - [Mutable variables](#mutable-variables)
    - [Constants](#constants)
    - [Shadowing](#shadowing)
  - [Data Types](#data-types)
    - [Scalar Types](#scalar-types)
      - [Integer](#integer)
        - [Integer Overflow](#integer-overflow)
      - [Floating-Point Types](#floating-point-types)
      - [Boolean Type](#boolean-type)
      - [Character Type](#character-type)
    - [Compound Types](#compound-types)
      - [Tuple Types](#tuple-types)
      - [Array Types](#array-types)
  - [Functions](#functions)
    - [Statements and Expressions](#statements-and-expressions)
    - [Function Return](#function-return)
  - [Control Flow](#control-flow)
    - [If](#if)
    - [Loop](#loop)
      - [Infinite print loop](#infinite-print-loop)
      - [Loop labels](#loop-labels)
      - [Return from loop](#return-from-loop)
    - [While](#while)
    - [For](#for)
<!--toc:end-->

<!-- prettier-ignore-end -->

## Variables and Mutability

### immutable variables

```rust
let x = 5;
```

### Mutable variables

```rust
let mut x = 5;
println!("{}", x); // 5
x = 6;
println!("{}", x); // 6
```

### Constants

Constants can be assigned expressions that can be evaluated during compile time, not result of a value computed at runtime.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

### Shadowing

When using `let`, variable is immutable and cannot be assigned, but we can perform a few transformations on a value and have the value be immutable after transformations.

```rust
let x = 5;
let x = x + 1;
{
    // inner scope
    let x = x + 2;
    println!("x"); // 12
}
println!("x"); // 6
```

Shadowing effectively creates new variable with the same name when using `let` again.

```rust
let spaces = "    ";
let spaces = spaces.len();
```

This spares us from coming up with different variable names such as `spaces_len`. If we use `let mut` in this case, the code will not compile due to type mismatch:

```rust
let mut spaces = "    ";
spaces = spaces.len(); // expected `&str`, found `usize`
```

## Data Types

### Scalar Types

#### Integer

Integer types (default `i32`):

| Length  | Signed  | Unsigned |
| ------- | ------- | -------- |
| 8-bit   | `i8`    | `u8`     |
| 16-bit  | `i16`   | `u16`    |
| 32-bit  | `i32`   | `u32`    |
| 64-bit  | `i64`   | `u64`    |
| 128-bit | `i128`  | `u128`   |
| arch    | `isize` | `usize`  |

Integer literals:

| Numbers Literals  | Examples      |
| ----------------- | ------------- |
| Decimal           | `98_222`      |
| Hex               | `0xff`        |
| Octal             | `0o77`        |
| Binary            | `0b1111_0000` |
| Bytes(`u8` only ) | `b'A'`        |

##### Integer Overflow

When compiling in debug mode, overflow will cause panic. When compiled with `--release` flag, overflow results in two's complement wrapping.
There are families of methods in standard library to handle overflow for primitive numeric types.

#### Floating-Point Types

Two floating-point types are `f32` and `f64` (default).

#### Boolean Type

```rust
let t = true;
let f: bool = false;
```

#### Character Type

```rust
let c = 'z';
let z = 'ℤ';
let heart_eyed_cat = '😻';
```

### Compound Types

#### Tuple Types

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
let (x, y, z) = tup; // Unpack tuple: x = 500, y = 6.4, z = 1
let first = tup.0 // 500
```

#### Array Types

Array has fixed number of elements.

```rust
let a = [1, 2, 3, 4, 5];
let b: [i32; 5] = [1, 2, 3, 4, 5];
let c = [3; 5]; // Same as [3, 3, 3, 3, 3];
let first = a[0] // 1
```

## Functions

### Statements and Expressions

```rust
let y = 6; // Statements
let x = (let y = 6) // let y = 6 does not returna value, so this will not compile
let z = {
    let w = 3;
    w + 1 // Expressions do not include ending semicolons
}; // z = 4
```

### Function Return

Function return value is synonymous with the value of final expression in function block. We can return early using `return` keyword.

```rust
fn main() {
    println!(plus_one(4)) // 5
}
// Function with return type
fn plus_one(x: i32) -> i32 {
    x + 1 // This expression value is returned
}
```

## Control Flow

### If

```rust
if number % 4 == 0 {
    println!("number is divisible by 4");
} else if number % 3 == 0 {
    println!("number is divisible by 3");
} else if number % 2 == 0 {
    println!("number is divisible by 2");
} else {
    println!("number is not divisible by 4, 3, or 2");
}

let condition = true;
let number = if condition { 5 } else { 6 };

let number2 = if condition { 5 } else { "six" }; // Cannot compile due to type mismatch
```

### Loop

#### Infinite print loop

```rust
loop {
    println("Infinite loop")
}
```

#### Loop labels

```rust

// loop label allow breaking/continuing outer loops
let mut first = 0;
let mut second = 0;
'increment_first': loop {
    loop {
        if second == 2 {
            break;
        }
        if first == 2 {
            break 'increment_first';
        }
        second += 1;
    }
    first += 1;
}
```

#### Return from loop

```rust
let mut counter = 0;
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;
    }
}; // 20
```

### While

```rust
let mut number = 3;
let mut result = 1;
while number != 0 {
    result *= number;
    number -= 1;
}
println!(result); // 6
```

### For

```rust
let a = [1, 2, 3, 4, 5];
for element in a {
    println!(element);
}

for number in (1..4).rev() {
    println!(number); // 4, 3, 2, 1
}
```
