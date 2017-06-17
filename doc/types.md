
## Primitives

### Integers

Ante supports 5 signed and unsigned integer primitives. 4 are platform-independent, sized at
8, 16, 32, and 64 bits respectively, of the form `[iu](8|16|32|64)`, where `i` indicates signed
and `u` indicates unsigned.. The last is sized to the platform bit-size i.e. 32 bits on a 32-bit 
platform, and so on: these are of the form `[iu]sz`.

### Characters

There are two character primitives: `c8`, for a byte character, and `c32` for a four-byte character.

### Floats

There are three float primitives: `f16`, `f32`, and `f64`.

### Booleans

Ante supports booleans with the `bool` type.
