# Stainless [![Build Status](https://travis-ci.org/reem/stainless.svg?branch=master)](https://travis-ci.org/reem/stainless)

> Stainless is a lightweight, flexible, unopinionated testing framework.

**Note that stainless currently requires the nightly version of the Rust compiler!**

## Installation

Add stainless as a dependency in your `Cargo.toml` file
``` toml
[dev-dependencies]
stainless = "*"
```

Add the following lines to the top of your
[root module](https://doc.rust-lang.org/book/crates-and-modules.html).
That file is normally called `src/main.rs` for executables and
`src/lib.rs` for libraries:

``` rust
#![feature(plugin)]
#![cfg_attr(test, plugin(stainless))]
```

This will make stainless available when you run the tests using `cargo
test`.

## Overview

Stainless exports the `describe!` syntax extension, which allows you
to quickly generate complex testing hierarchies and reduce boilerplate
through `before_each` and `after_each`.

Stainless currently supports 4 types of subblocks:
 - `before_each` and `after_each`,
 - `it` and `failing`
 - `bench`
 - nested `describe!`

`before_each` and `after_each` allow you to group common
initialization and teardown for a group of tests into a single block,
shortening your tests.

`it` generates tests which use `before_each` and `after_each`.
`failing` does the same, except the generated tests are marked with
`#[should_panic]`.

`bench` allows you to generate benchmarks in the same fashion, though
*`before_each` and `after_each` blocks do not currently affect `bench`
blocks*.

Nested `describe!` blocks allow you to better organize your tests into
small units and gives you granular control over where `before_each`
and `after_each` apply. Of course the `before_each` and `after_each`
blocks of the wrapping `describe!` blocks are executed as well.

Together, these 4 types of subblocks give you more flexibility and
control than the built in testing infrastructure.

## Example

```rust
describe! stainless {
    before_each {
        // Start up a test.
        let mut stainless = true;
    }

    it "makes organizing tests easy" {
        // Do the test.
        assert!(stainless);
    }

    after_each {
        // End the test.
        stainless = false;
    }

    bench "something simple" (bencher) {
        bencher.iter(|| 2 * 2)
    }

    describe! nesting {

        before_each {
          let mut inner_stainless = true;
        }

        after_each {
          inner_stainless = false;
        }

        it "makes it simple to categorize tests" {
            // It even generates submodules!
            assert_eq!(2, 2);
        }
    }
}
```

Expands to (roughly):

```rust
mod stainless {
    #[test]
    fn makes_organizing_tests_easy() {
        let mut stainless = true;
        assert!(stainless);
        stainless = false;
    }

    #[bench]
    fn something_simple(bencher: &mut test::Bencher) {
        bencher.iter(|| 2 * 2)
    }

    mod nesting {
        #[test]
        fn makes_it_simple_to_categorize_tests() {
            let mut stainless = true;
            let mut inner_stainless = true;
            assert_eq!(2, 2);
            inner_stainless = false;
            stainless = false;
        }
    }
}
```

## Importing modules

At this point it is not possible to put `use` statements inside the
`describe!` blocks. To allow usage of data structures from other
modules and crates each `describe!` block comes with a silent `pub use
super::*;` in it. That way everything you `pub use` in the containing
module is available in your tests.

```rust
#[cfg(test)]
mod tests {
    pub use std::collections::HashMap;

    describe! stainless {
        it "can use HashMap" {
            let map = HashMap::new();
        }
    }
}
```

## License

MIT
