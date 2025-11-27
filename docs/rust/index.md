# Rust 

 Rust focus on safety, performance and concurrency
safe and fast: reliability of high-level languages with the speed of low-level ones

- Cargo: build system and package manager
- rustfmt: it formats code for consistency and readability
- rust-analyzer: IDE tool offering code completion and inline errors

https://rust-lang.org/tools/install/  
brew install rust

file name in snake_case

marcro end by "!"

```rust
fn main() {
    println!("Hello, world!");
}
```
The code above defines the main function, which serves as the entry point for every Rust executable program. Inside the main function, the println! macro (identified by the exclamation mark) outputs the string "Hello, world!" to the console.


rustfmt main.rs
This command automatically formats your Rust code according to community style guidelines.


## Comment

```rust
// single comment line.
```

```rust
/*
 * Multi-line comment.
 * Spanning multiple lines.
 */
```

## Variables and mutability

```rust
fn main() {
    let y = 10;
    println!("The value of y is: {}", y);
    y = 20; // This line will cause a compilation error
    println!("The value of y is: {}", y);
}
```

```rust
fn main() {
    let mut y = 10;
    println!("The value of y is: {}", y);
    y = 20;
    println!("The value of y is: {}", y);
}
```

## Scalar and Compounds data types

### Scalar data type:

  - integers: (default i32)
    - signed(i): can be positive +ve or negative -ve
    - unsigned(u): only positive +ve
  - floating-point numbers:
    - f32
    - f64 -default-
  - booleans:
    - true
    - false
  - characters:
    - z, Z and 😻

```shell
+------------+----------+-----------+-----------------------------------------------+
|  Type      | Signed?  |  Bits     | Value Range                                   |
+------------+----------+-----------+-----------------------------------------------+
|   i8       | Yes      |   8-bit   |  -128 to 127                                  |
|   u8       | No       |   8-bit   |     0 to 255                                  |
+------------+----------+-----------+-----------------------------------------------+
|   i16      | Yes      |  16-bit   |  -32,768 to 32,767                            |
|   u16      | No       |  16-bit   |     0 to 65,535                               |
+------------+----------+-----------+-----------------------------------------------+
|   i32      | Yes      |  32-bit   |  -2,147,483,648 to 2,147,483,647              |
|   u32      | No       |  32-bit   |     0 to 4,294,967,295                        |
+------------+----------+-----------+-----------------------------------------------+
|   i64      | Yes      |  64-bit   |  -9.22e18 to 9.22e18                          |
|   u64      | No       |  64-bit   |     0 to 1.84e19                              |
+------------+----------+-----------+-----------------------------------------------+
|   i128     | Yes      | 128-bit   |  -1.70e38 to 1.70e38                          |
|   u128     | No       | 128-bit   |     0 to 3.40e38                              |
+------------+----------+-----------+-----------------------------------------------+
|   isize    | Yes      | pointer   |  Depends on architecture (32-bit or 64-bit)   |
|   usize    | No       | pointer   |  Depends on architecture (32-bit or 64-bit)   |
+------------+----------+-----------+-----------------------------------------------+
```

### Compound data type

Compound types group multiple values into one type. Rust provides two primitive compound types: tuples and arrays.

#### Tuples
Tuples allow you to combine values of different types into a single compound type. Once declared, the length of a tuple cannot change.

Consider the following example, where a tuple is declared with an i32, an f64, and a u8:

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);


    // Destructuring the tuple into individual variables
    let (x, y, z) = tup;
    println!("The value of y is: {}", y);


    // Accessing tuple elements directly using dot notation
    let five_hundred = tup.0;
    let six_point_four = tup.1;
    let one = tup.2;
}
```

#### Arrays
Arrays in Rust are collections of values of the same type with a fixed length. Array indexing starts at 0 and goes up to the length of the array minus one.

Below is an example of creating an array and accessing its elements:

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];


    let first = a[0];
    let second = a[1];
    println!("The first element is: {}", first);
    println!("The second element is: {}", second);
}
```

constants always immutuable
const naming conventions always in maj like

const DEFAULT_TIMEOUT: u32 = 30;