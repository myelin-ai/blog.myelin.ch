---
layout: post
title:  "Announcing Mockiato - A strict, yet friendly mocking library for Rust 2018"
author: Jeremy, Ruben, Jan, Mathias
---

> ⚠️ Requires the nightly compiler

We're proud to announce [mockiato](https://github.com/myelin-ai/mockiato)! For the last few months, we tackled the issue of creating a usable mocking library.
Our primary goals were
- Ease of use: The mocks are written in idiomatic Rust and don't rely on custom macro syntax.
- Maintainability: The entire code base strives to follow the rules of Clean Code and Clean Architecture as specified by Robert C. Martin.
- Strict expectation enforcement: Mockiato catches unexpected behavior as soon as it happens instead of returning default values.

Relying on mocks instead of implementations allows you to test components in isolation of their dependencies, providing real unit testing capabilities.

Mockiato enjoys a broad test suite, composed of many unit tests and [integration](https://github.com/myelin-ai/mockiato/tree/master/tests) [tests](https://github.com/myelin-ai/mockiato/tree/master/crates/mockiato-compiletest/tests/ui).

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
| At most (inclusive)   | `RangeToInclusive` | `..=3`  |
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

| Feature | Description | Tracking Issue |
| ---- | ----------- | -------------- |
| `Sync` / `Send` | Mocks are neither sync, nor send. | [#106](https://github.com/myelin-ai/mockiato/issues/106) |
| Lifetimes on traits | Lifetimes are supported in the method signature, but not on the trait itself. | [#117](https://github.com/myelin-ai/mockiato/issues/117) |
| Reference to generics | References to types containing a generic type are currently not supported due to a limitation in codegen involving lifetimes. | [#123](https://github.com/myelin-ai/mockiato/issues/123) |
| Stable support | Mockiato requires nightly because we use the unstable `proc_macro_diagnostics` API to print helpful messages. Printing expected calls requires the unstable feature `specialization`. | [#161](https://github.com/myelin-ai/mockiato/issues/161) |
