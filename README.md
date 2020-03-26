# `fmt_adapter`
![Crates.io link](https://img.shields.io/crates/v/fmt_adapter "fmt_adapter on Crates.io")

This crate provides newtype adaptors to and from any formatting trait. More specifically, it allows one to wrap a value into a filter wrapper which filters out the implementations of all formatting traits except for one and then wrap it into a adaptor wrapper which implements one specific formatting trait, which can be different from the original formatting trait. This is better demonstrated with an example:
```rust
// Let's create an example type...
#[derive(Copy, Clone, Debug)]
enum Sign {
    Positive,
    Negative
}
//...and implement a formatting trait by hand.
impl fmt::Display for Sign {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}",
            match self {
                Positive => '+',
                Negative => '-'
            }
        )
    }
}

let sign = Sign::Positive;
println!("The sign character is {}", sign); // Outputs the Display formatting
println!("The sign debug form is {:?}", sign); // Outputs the derived Debug formatting
let sign = DebugFromDisplay(sign); // Wrap it into a formatting adapter
println!("The sign debug form is {:?}", sign); // Outputs the Display formatting, even
                                               // though we have {:?} as the formatting mode
let sign = UpperHexFromDebug(sign.0); // Get the value from the previous adapter
                                      // and do something very random
println!("The sign in uppercase hexadecimal is `0x{:X}`", sign);
```
All adapters in the crate are generated from a list of traits using a build script. As such, no manual editing of the adaptor delarations and their `impl` blocks is possible. The generated files are listed in `.gitignore` — use [`cargo download`][cargo-dl] and build the crate if you want to explore the generated results, or simply take a look at the `build.rs` file.

The crate is `#![no_std]`, meaning that it works in a freestanding context and only depends on `core::fmt`, which requires a functional global allocator.

## Types of adapters
Here's a list of adapter types (values between `{}` can be replaced by any formatting trait):
- `Only{trait}` — filters out all formatting traits of a type except for `{trait}`. This can be used to restrict access to formatting traits when transferring objects between subsystems.
- `{out_tr}From{in_tr}` — implements `{out_tr}` by using the results of formatting with `{in_tr}`. This can be used to provide arbitrary formatting methods to interfaces which only accept one specific formatting method while still correctly implementing other formatting traits exactly as specified by the `fmt` documentation.

[cargo-dl]: https://crates.io/crates/cargo-download "cargo-download — a cargo subcommand for downloading crates from crates.io"
