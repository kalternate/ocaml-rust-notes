# How Rust was influenced by ML-languages like OCaml


## What is OCaml?

OCaml is a statically-typed, multi-paradigm programming language. It's fairly similar to a language your probably familiar with: Haskell. It supports functional, imperative, and even object-oriented programming. The 'O' in OCaml comes from its former name, Objective Caml, as the language was original an extension of Caml that added objects. It's 1.0 release was in 1996.

```ocaml
let rec fib m n i =
    if i < 1 then m
    else fib n (n + m) (i - 1)

let fib = fib 0 1

let () =
  Printf.printf "fib(%d) = %d\n" 20 (fib 20)
  (* fib(20) = 6765 *)
```

## What is Rust?

Rust is a statically-typed, multi-paradigm programming language. It supports both imperative and functional programming. Rust took a ton of inspiration from ML-languages (as we'll see) and the first Rust compiler was actually written in OCaml. The annual Stack Overflow Developer Survey has consistently found Rust to be the most loved programming language among developers who use it. It's 1.0 release was in 2015. According to the [Rust Reference](https://doc.rust-lang.org/reference/influences.html), Rust was inspired by four ML-language features: pattern matching, algebraic data types, type inference, and semicolon statement separation. 

```rust
fn fib_rec(m: u32, n: u32, i: u32) -> u32 {
    if i < 1 {
        m
    } else {
        fib_rec(n, n + m, i - 1)
    }
}

fn fib(i: u32) -> u32 {
    fib_rec(0, 1, i)
}


fn main() {
    println!("fib({}) = {}", 20, fib(20));
    // fib(20) = 6765
}
```

## Pattern Matching

Pattern matching exists in OCaml and Rust in several forms. For instance, unwrapping a tuple as shown below.

```ocaml
let symbol (s, n, w) = s
let atomic_number (s, n, w) = n
let mercury = ("Hg", 80, 200.592)

let () = Printf.printf "%s's atomic number is %d\n" (symbol mercury) (atomic_number mercury)
(* Hg's atomic number is 80 *)

```

Both OCaml and Rust have a `match` control flow construct which allow the programmer to define distinct patterns a value may match with. See the following examples using option and result types.

### Ocaml
```ocaml
(* float_of_string_opt returns a float option type which is checked using pattern matching. *)
let print_number num_str = match float_of_string_opt num_str with
  | Some num -> Printf.printf "Your number is %f\n" num 
  | None -> Printf.printf "'%s' is not a number\n" num_str

let () = print_number "42"
(* Your number is 42.000000 *)

let () = print_number "duck"
(* 'duck' is not a number *)
```
OCaml has special pattern matching for lists. Rust doesn't have this.
```ocaml
let rec print_list list = match list with
  | head :: tail -> let () = Printf.printf "%d " head in print_list tail
  | [] -> ()

let () = print_list [1; 2; 3]
(* 1 2 3 *)
```

### Rust
```rust
use std::str::FromStr;

fn print_number(num_str: &str) {
    match f32::from_str(num_str) {
        Ok(num) => println!("Your number is {}", num),
        Err(_) => println!("'{}' is not a number", num_str)
    }
}


fn main() {
    print_number("42");
    // Your number is 42
    
    print_number("duck");
    // 'duck' is not a number
}
```

Rust lets you do pattern matching in `if` and `while` expressions as well.

```rust
fn main() {
    let mut birds = vec!["goose!","duck","duck"];
    
    // pop() returns an Option<&str> type 
    while let Some(bird) = birds.pop() {
        print!("{} ", bird)
    }
    // duck duck goose!
}
```

Rust allows for nested type patterns in a single match expression.

```rust
use std::str::FromStr;

fn main() {
    // get returns an Option<&str> type. map(i32::from_str) maps the interior 
    // string slice to a result that may contain an int.
    match "1234".get(0..2).map(i32::from_str) {
        Some(Ok(num)) => assert_eq!(num, 12),
        _ => panic!()
    }
}
```

## Algebraic Data Types
Algebraic data types are types composed of other types. There are two flavors of ADTs: product types (a.k.a structs, records) and sum types (a.k.a enums, variants). OCaml and Rust have both flavors. 

### Product Types
In OCaml, product types are called records. In Rust, they are called structs. Product types are a collection of fields, with each field having a name and/or an index, as well as an associated type. Tuples are also product types.

**OCaml**
```ocaml
type element = {
  symbol: string;
  atomic_number: int;
  weight: float;
}

(* mercury is of type element *)
let mercury = {
  symbol = "Hg";
  atomic_number = 80;
  weight = 200.59;
}

let () = assert (mercury.atomic_number == 80)
```

**Rust**
```rust
struct Element {
    symbol: String,
    atomic_number: u32,
    weight: f32
}

fn main() {
    let iron = Element{
        symbol: String::from("Fe"),
        atomic_number: 26,
        weight: 55.845
    };

    assert_eq!(iron.atomic_number, 26);
}
```

Rust also has syntax for structs with unnamed fields, which are basically named tuples.

```rust
struct FullName(String, String);

fn main() {
    let first_name = String::from("Keagan");
    let last_name = String::from("Edwards");
    let full_name = FullName(first_name.clone(), last_name.clone());
    assert_eq!(first_name, full_name.0);
    assert_eq!(last_name, full_name.1);
}
```

### Sum Types
Sum types are types that can be one of several different enumerated variants. In OCaml, sum types are called variants. In Rust, they are called enums.

**OCaml**
```ocaml
(* This is a sum type*)
type duck_flavor = 
  | Mallard
  | Perkin
  | Wood
  | Rubber

(* This is a product type that contains a sum type *)
type duck = {
  name: string;
  flavor: duck_flavor;
}
```
**Rust**
```rust
// This is a sum type
enum DuckFlavor {
    Mallard,
    Perkin,
    Wood,
    Rubber
}

// This is a product type that contains a sum type
struct Duck {
    name: String,
    flavor: DuckFlavor
}
```

### Sum Types w/ Fields
OCaml and Rust both allow specific variants of sum types to contain data. Sum types of with fields are sometimes called variant records or tagged unions. While lots of programming languages have some form of simple sum type, sum types which contain data are pretty rare. As we'll see, these kind of types are what allow pattern matching to be so powerful.

**OCaml**
```ocaml
(* We kept changing how to uniquely identify users *)
type user_id = 
  | Number of int
  | Email of string
  | Full_name of string * string
  | Missing
```

**Rust**
```rust
enum UserId {
    Number(i32),
    Email(String),
    FullName(String, String),
    Missing
}
```

This example might seem contrived but this is a criminally underrated language feature. This is what is used to create option and result types, and we can also use it to compose complex data structures. 

### Binary Tree Example

**OCaml**
```ocaml
type 'a tree = 
 | Node of 'a * 'a tree * 'a tree
 | Empty

let rec add_node item tree = match tree with
  | Node (other_item, left, right) -> if item < other_item 
    then Node (other_item, add_node item left, right) 
    else Node (other_item, left, add_node item right) 
  | Empty -> Node (item, Empty, Empty)

let rec print_tree tree = match tree with
  | Node (item, left, right) -> 
    let () = print_tree left in 
    let () = Printf.printf "%d " item in 
    print_tree right
  | Empty -> ()

let tree = add_node 3 Empty
let tree = add_node 7 tree
let tree = add_node 1 tree
let tree = add_node 5 tree
let tree = add_node 2 tree
let tree = add_node 6 tree
let tree = add_node 4 tree
let () = print_tree tree
(* 1 2 3 4 5 6 7 *)
```

**Rust**
```rust
use std::fmt::Display;

enum Tree<T> {
    Node(T, Box<Tree<T>>, Box<Tree<T>>),
    Empty
}

fn add_node<T: PartialOrd>(item: T, tree: Tree<T>) -> Tree<T> {
    match tree {
        Tree::Node(other_item, left, right) => {
            if item < other_item {
                Tree::Node(other_item, Box::new(add_node(item, *left)), right)
            } else {
                Tree::Node(other_item, left, Box::new(add_node(item, *right)))
            }
        },
        Tree::Empty => {
            Tree::Node(item, Box::new(Tree::Empty), Box::new(Tree::Empty))
        }
    }
}

fn print_tree<T: Display>(tree: Tree<T>) {
    match tree {
        Tree::Node(item, left, right) => {
          print_tree(*left);
          print!("{} ", item);
          print_tree(*right);
        },
        Tree::Empty => {}
    }
}

fn main() {
    let tree = add_node(3, Tree::Empty);
    let tree = add_node(7, tree);
    let tree = add_node(1, tree);
    let tree = add_node(5, tree);
    let tree = add_node(2, tree);
    let tree = add_node(6, tree);
    let tree = add_node(4, tree);

    print_tree(tree)
    // 1 2 3 4 5 6 7
}
```

## Type Inference
Even though both OCaml and Rust are statically typed, we haven't seen many type annotations in these examples. Both OCaml and Rust can infer the type of value by examining the abstract syntax tree of the program. One difference is that Rust needs type annotations in function signatures.  

```rust
fn main() {
    // Rust can figure out optional duck is of type Option<&str>
    let optional_duck = Some("duck");
    // Here we need a type annotion becuase Rust cannot figure out what kind of option type is it.
    let optional_goose: Option<&str> = None;

}
```

## Semicolon Statement Separation
All statements and blocks in Rust are actually expressions. By default, they return the unit type `()`. Semicolons are used to separate statements .

```rust
fn main() {
    let mut x = 0;

    // Even loops in Rust can be evaluated as expressions.
    let y = loop {
        if x == 42 {
            break x;
        } else {
            x +=1;
        };
    };
    
    assert_eq!(x, y);
}
```

## Differences between Rust & OCaml
We've seen a lot of the ways Rust was inspired by OCaml, but there are lots of differences between the languages as well.

* Syntax is very different! OCaml syntax is similar to other ML-languages while Rust is a lot more C-like.
* Memory Management! OCaml uses garbage collection; memory management in Rust could be its whole own presentation. It doesn't use garbage collection, which comes with lots of trade-offs!
* Exceptions! Even though OCaml has the result type, it also has exceptions. Rust does not have exceptions at all (because they're bad) and just uses the result type for error handling.
* Number of string types! OCaml has one string type (as far as I can tell) whereas Rust has around 10 depending on how you count.


