# What the Fix Was

The fix addresses a fundamental misunderstanding of how the **SetColor (0x04)** command's data length is encoded in SC3 scripts. The Rust-based `sc3tools` incorrectly handled color data:

**The Bug:** The original code assumed SetColor always uses either:
- A simple 1-byte value (X360 games), or
- A full "expression" with a 0x00 terminator (Steam games)

The Rust `sc3tools` (`sc3tools/src/sc3.rs`) parses SetColor using `Expr::parse`, which treats the color data as a generic expression terminated by `0x00`. This **double-counts the 0x00 terminator**, causing "expected more input" errors. The older summary (part of a previous run) explicitly states: *"Rust's `Expr::parse` is independently broken (double-counts the `0x00` terminator → 'expected more input')."*

### The Rust sc3tools Bug (fixed)
In `sc3tools/src/sc3.rs` (Line 149):
```rust
0x04 => parse(i, Expr::parse, StringToken::Color),
```

And `Expr::parse` (Lines 95-99):
```rust
pub fn parse(i: &'a [u8]) -> IResult<&'a [u8], Self> {
    map(recognize(many_till(Self::token, tag(&[0x00u8]))), |slice| {
        Expr(Cow::from(slice))
    })(i)
}
```

The `many_till(Self::token, tag(&[0x00u8]))` reads tokens until it finds a `0x00` byte, but the `0x00` terminator is included in the recognized slice. The `Expr::parse` then expects more input after the terminator (since `recognize` consumes it but `many_till` stops at it without consuming), causing the "expected more input" error.

### The implemented fix:
**SetColor color fix** (`sc3.rs`):
- `StringToken::Color` now stores `Cow<'a, [u8]>` (raw bytes) instead of `Expr<'a>`
- New `parse_color` function: reads first byte, if `< 0x80` → 1 byte (X360), if `>= 0x80` → `const_len + 1` bytes including terminator (Steam)
- Updated `coz.rs` serialize/deserialize to handle raw bytes
