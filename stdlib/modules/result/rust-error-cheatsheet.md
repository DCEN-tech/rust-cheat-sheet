# Rust Error Handling Cheatsheet - Result handling functions

---

## RESULT

### Definition

*Result<T, E>* is the type used for returning and propagating errors.  
It is an enum with the variants:

   * *Ok(T)*
      representing success and containing a value
   
   * *Err(E)*
      representing error and containing an error value

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```


### Using Result

A simple function returning Result might be defined and used like so:

```rust
fn div(n1: isize, n2: isize)
   -> Result<isize, &'static str>
{
   match n2 {
      0 => Err("Division by 0 !"),
      _ => Ok(n1 / n2)
   }
}

fn main() 
{
   let inputs = [(4, 2), (6, 0)];

   for input in inputs {
      let n1 = input.0;
      let n2 = input.1;
      match div(n1, n2) {
         Ok(r)  => println!("Division: {} / {} = {}", n1, n2, r),
         Err(e) => println!("Error: {}", e),
      }
   }
}
```

### Result Methods

Pattern matching on Results is clear and straightforward for simple cases, but Result comes with some convenience 
methods that make working with it more succinct:

   ---
   * [and](#method_and)
   * and_then
   * as_deref
   * as_deref_mut
   * as_mut
   * as_ref
   * cloned
   * cloned
   * copied
   * copied
   * err
   * expect
   * expect_err
   * flatten
   * inspect
   * inspect_err
   * into_err
   * into_ok
   * is_err
   * is_err_and
   * is_ok
   * is_ok_and
   * iter
   * iter_mut
   * map
   * map_err
   * map_or
   * map_or_else
   * ok
   * or
   * or_else
   * transpose
   * unwrap
   * unwrap_err
   * unwrap_err_unchecked
   * unwrap_or
   * unwrap_or_default
   * unwrap_or_else
   * unwrap_unchecked
   ---

#### and {#method_and}

> <u>Syntax</u>: 
```rust
fn and<U>(self, res: Result<U, E>) -> Result<U, E>
```
>
> <u>Returns</u>:
>  * *res* if the result is *Ok*
>  * otherwise returns the Err value of self
>
> Arguments passed to and are eagerly evaluated; if you are passing the result of a function call, it is recommended to 
> use *and_then*, which is lazily evaluated.

> <u>Example:</u>

```rust
let x: Result<u32, &str> = Ok(2);
let y: Result<&str, &str> = Err("late error");
assert_eq!(x.and(y), Err("late error"));

let x: Result<u32, &str> = Err("early error");
let y: Result<&str, &str> = Ok("foo");
assert_eq!(x.and(y), Err("early error"));

let x: Result<u32, &str> = Err("not a 2");
let y: Result<&str, &str> = Err("late error");
assert_eq!(x.and(y), Err("not a 2"));

let x: Result<u32, &str> = Ok(2);
let y: Result<&str, &str> = Ok("different result type");
assert_eq!(x.and(y), Ok("different result type"));
```

---

   * and_then
   * as_deref
   * as_deref_mut
   * as_mut
   * as_ref
   * cloned
   * cloned
   * copied
   * copied
   * err
   * expect
   * expect_err
   * flatten
   * inspect
   * inspect_err
   * into_err
   * into_ok
   * is_err
   * is_err_and
   * is_ok
   * is_ok_and
   * iter
   * iter_mut
   * map
   * map_err
   * map_or
   * map_or_else
   * ok
   * or
   * or_else
   * transpose
   * unwrap
   * unwrap_err
   * unwrap_err_unchecked
   * unwrap_or
   * unwrap_or_default
   * unwrap_or_else
   * unwrap_unchecked








## Concepts

### Handling Errors in Rust

Functions return Result whenever errors are expected and recoverable.  
In the std crate, Result is most prominently used for I/O.

Normal way to handle errors we use the match statement:



## Method chaining in error handling

If all the errors are wrapped to match statement, we get soon plenty of match
statements inside each other.

One alternative to address this is to chain methods to handle the errors.

```rust
value = function_to_do_nice_things()
  .and_then(other_function)
  .map_err(|e| module_error_from_io_error(e))?
```

Example calls two functions, gets error value from the first that fails, maps it from io error to
our own error and returns it from the function to the caller. If calls are ok, we unwrap the
Ok result and assign it to variable value.

It is pretty difficult to find the right error mapping funtion. There are 21 of them
and the descriptions are pretty cryptic.

## Cheatsheet

This cheatsheet lists 21 error Result handling functions and tells what they do.

The division is
* Six functions that map results to results
* Two functions that map results to values (or results)
* Eight functions that can be used to extract Ok value from the Result or to get information on existence of Ok result
* Four functions that can be used to extract Err value or to get information of existence of Err result
* One special conversion function

The first column tells what is done to the result if it is Ok variant. The second column tells what is done to the Err variant.
This hopefully makes it easier to find the right function for specific purpose.

For example, if you are looking for a function that leaves Err result as is and maps the Ok result with a function, you quickly
find that map and and_then are such functions. Then you decide if you want to map simply with a function mapping the value (map)
or if you want to return a full Ok/Err result with and_then function.

## In these examples
* r is the result what these functions address
* t is the Ok value inside r, Ok(t)
* e is the Err value inside r, Err(e)
* r2 is the second result given as an argument having t2 and e2
* f is a function that gets t as input and generates t'
* F is a function that gets t as input and generates new Result(t', e')
* g is a function that gets e as input and generates e'

## Mapping result to result

| Ok(t) -> ?                    | Err(e) -> ?                                 | Code r:                                            | Description
|-------------------------------|---------------------------------------------|----------------------------------------------------|----------
| t -> Ok(t')                   | Unchanged                                   | <code>r.map(\|t\| f(t))</code>                     | Map ok with function, error as is, mapping can not result error
| t -> (t', e')                 | Unchanged                                   | <code>r.and_then(\|t\| F(t))</code>                | Calls function for Ok value and propagates errors. When you chain these like r.and_then().and_then(), it returns result of last function or the first happened error.
| Unchanged                     | _e -> (t', e')                              | <code>r.or_else(\|_e\| F())</code>                 | In chain r.or_else(f1).or_else(f2) calls functions until one succeeds, does not call after first success, argument must return Result type. Called function gets the error value as argument but likely do not use it.
| Unused, return arg r2 (t2, e2)| Unchanged                                   | <code>r.and(r2)</code>                             | In chain r.and(r2).and(r3) return last ok result or first error
| Unchanged                     | Unused, return arg r2 (t2, e2) instead      | <code>r.or(r2)</code>                              | In chain r.or(r2).or(r3) return value of first Ok or last error, evaluates all or values
| Unchanged                     | e -> Err(e')                                | <code>r.map_err(\|e\| g(e))</code>                 | Map error with g(e), that return a normal type that is automatically converted to error result. Map function can not return an error.

## Mapping Result to any type

| Ok -> ?                       | Err -> ?                                    | Code                                               | Description
|-------------------------------|---------------------------------------------|----------------------------------------------------|----------
| t -> t' (returned as is)      | e -> e' (returned as is)                    | <code>r.map_or_else(\|e\| g(e), \|t\| f(t))</code> | Map both Ok and Err with a function. Result can be of any type but it has to be same for both Ok and Error. Err mapping function is first because it is considered as a "default value if normal processing fails" like in the map_or.
| t -> t' (returned as is)      | Literal (returned as is)                    | <code>r.map_or(literal, \|t\| f(t))</code>         | Map with function. If error, use literal as a default value. Mapping function can return Result but also any other type that matches literal. Note that this does NOT meant that if mapping function fails, use literal. It means that if we can not use mapping function due to error, give the literal instead.

## Extract Ok value

| Ok -> ?                       | Err -> ?                                    | Code                                           | Description
|-------------------------------|---------------------------------------------|------------------------------------------------|----------
| t                             | stop function and return Err(e) immediately | <code>r?</code>                                | If error, return from the function using this same result. Function result must be compatible.
| t                             | panic                                       | <code>r.unwrap()</code>                        | Panics with error, may use e as panic message.
| t                             | panic with message                          | <code>r.expect("string")</code>                | unwrap() with a given panic message.
| t                             | Literal as t'                               | <code>r.unwrap_or(literal)</code>              | Unwrap, if error, use literal from arguments instead.
| t                             | e -> t'                                     | <code>r.unwrap_or_else(\|e\| g(e))</code>      | Extract value or derive it from error with function
| t                             | Default as t'                               | <code>r.unwrap_or_default()</code>             | Returns value or default for that type (if set)
| true                          | false                                       | <code>r.is_ok()</code>                         | True if ok
| Option::Some(t)               | Option::None                                | <code>r.ok()</code>                            | If Ok, return Option::Some(t), in case of error returns Option::None

## Extract error

| Ok -> ?                                      | Err -> ?                     | Code                                           | Description
|-------------------------------|---------------------------------------------|------------------------------------------------|----------
| panic                         | e                                           | <code>r.unwrap_err()</code>                    | Panics, may shows value of t
| panic                         | e                                           | <code>r.expect_err("message")</code>           | Panics if ok, with set panic message, prints value of t
| false                         | true                                        | <code>r.is_err()</code>                        | True if error
| None                          | Some(e)                                     | <code>r.err()</code>                           | Some(e) if error or None if no error

## Convert

| Ok -> ?                                      | Err -> ?                        | Code                                           | Description
|-------------------------------|---------------------------------------------|------------------------------------------------|----------
| t -> Some(t)                  | e -> Some(e)                                | <code>r.transpose()</code>                     | Take Option (especially Option::None) out from Result

## Question mark operator

To use r?, function must return compatible Result type. For testing,
the main function and tests can return Result type (Rust 2018)

## Own errors

It is customary to define your own Error type for your program
```
pub struct MyError {};
pub type Result<T> = result::Result<T, MyError>;
impl fmt::Display for MyError {
  ..
}
impl fmt::Debug for MyError {
  ..
}
```

## Generating results for testing etc.

```
let r: Result<u32, String> = Ok(233);
let s: Result<u32, String> = Err("meaningless input");
let t: Result<(), ()> = Ok(());
```
