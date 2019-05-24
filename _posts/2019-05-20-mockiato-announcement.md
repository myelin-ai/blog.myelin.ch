---
layout: post
title:  "Announcing Mockiato - A strict, yet friendly mocking library for Rust 2018"
author: Jeremy, Ruben, Jan
---

## Disclaimer
 > ⚠️ Requires the nightly compiler

## Table of Contents
- [The basics](#the-basics)  
- [Expecting a method call a certain amount of times](#expecting-a-method-call-a-certain-amount-of-times)
- [Specifiying return values](#specifying-return-values)
- [Limitations](#limitations)

## The Basics
Annotating a trait `Greeter` with `#[mockable]` generates a struct `GreeterMock` that implements `Greeter`
and provides an `expect_<method_name>` method for every method in the trait.

Calling these `expect` methods sets up expectations which panic when invoking an 
unexpected call or when `GreeterMock` goes out of scope with unmet expectations.

```rust
#[cfg(test)]
use mockiato::mockable;

#[cfg_attr(test, mockable)]
trait Greeter {
    fn greet(&self, name: &str);
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greet_the_world() {
        let mut greeter = GreeterMock::new();

        greeter.expect_greet(|arg| arg.partial_eq("world"));
        
        greeter.greet("world");
    }
}
```

## Expecting a method call a certain amount of times
By default, each expected method is configured to be called exactly once.

To be able to specify the number of expected calls, the generated `expect_` methods return a `MethodCallBuilder`, which amongst other methods contains a `times` method.

The `times` method accepts a usize value or a range. Here is the full list of supported values from the documentation:

| Description           | Type               | Example |
| --------------------- | ------------------ | ------- |
| Exact amount of times | `u64`              | `3`     |
| Any amount of times   | `RangeFull`        | `..`    |
| At least              | `RangeFrom`        | `3..`   |
| At most (exclusive)   | `RangeTo`          | `..3`   |
| At most (inclusive)   | `RangeToInclusive` | `..3`   |
| Between (exclusive)   | `Range`            | `3..4`  |
| Between (inclusive)   | `RangeInclusive`   | `3..=4` |

```rust
#[cfg(test)]
use mockiato::mockable;

#[cfg_attr(test, mockable)]
trait Greeter {
    fn greet(&self, name: &str);
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greet_the_world() {
        let mut greeter = GreeterMock::new();

        greeter
            .expect_greet(|arg| arg.partial_eq("world"))
            .times(1..2);
        
        greeter.greet("world");
    }
}
```

## Specifying return values
The `MethodCallBuilder` also provides a `returns` method.

This `returns` method allow you to specify a return value, which is returned when the corresponding mock method is invoked.

```rust
#[cfg(test)]
use mockiato::mockable;

#[cfg_attr(test, mockable)]
trait Greeter {
    fn greet(&self, name: &str) -> String;
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greet_the_world() {
        let mut greeter = GreeterMock::new();

        greeter
            .expect_greet(|arg| arg.partial_eq("world"))
            .returns(String::from("Hello world"));

        assert_eq!("Hello world", greeter.greet("world"));
    }
}
```

### Specifying different return values for the same call
For each method of the `Greeter` trait a corresponding `expect_<method_name>_calls_in_order` method is generated.

Ordered expectations can be used together with [`times`](#expecting-a-method-call-a-certain-amount-of-times).

```rust
#[cfg(test)]
use mockiato::mockable;

#[cfg_attr(test, mockable)]
trait Greeter {
    fn greet(&self, name: &str) -> String;
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greet_the_world() {
        let mut greeter = GreeterMock::new();

        greeter
            .expect_greet(|arg| arg.partial_eq("world"))
            .returns(String::from("Hello world"));

        greeter
            .expect_greet(|arg| arg.partial_eq("world"))
            .returns(String::from("Hi world"));

        greeter.expect_greet_calls_in_order();

        assert_eq!("Hello world", greeter.greet("world"));
        assert_eq!("Hi world", greeter.greet("world"));
    }
}
```

## Limitations
### `Sync` / `Send`
Mocks are neither sync, nor send. The tracking issue for this limitation is [#106](https://github.com/myelin-ai/mockiato/issues/106).

### Lifetimes on traits
Lifetimes are supported in the method signature, but not on the trait itself. The tracking issue for this limitation is [#117](https://github.com/myelin-ai/mockiato/issues/117).

### References to generics
References to types containing a generic type are currently not supported due to a limitation in codegen involving lifetimes. The tracking issue for this limitation is [#123](https://github.com/myelin-ai/mockiato/issues/123).

### Stable support
Mockiato requires nightly because we use the unstable `proc_macro_diagnostics` API to print helpful messages.
Printing expected calls requires the unstable feature `specialization`.
