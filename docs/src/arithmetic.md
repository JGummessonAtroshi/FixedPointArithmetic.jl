# Arithmetic model

Products are exact: the word and fraction lengths of the operands add. Sums keep the word and fraction lengths of
their operands, so no bits are rounded away and a carry out of the top bit goes through the overflow mode. This is
the model of hardware fixed-point types such as `ap_fixed` and `ac_fixed`, except that their sums grow by one bit.

| Operation | Result |
|:--|:--|
| `X(x)`, `convert(X, x)`, storing into an array of `X` | `x` rounded with the rounding mode of `X`, then brought into its range with its overflow mode. Never throws: NaN becomes 0, and so do ±Inf with `Wrap`. |
| `a + b`, `a - b`, `-a`, `abs(a)` of one format | the same format, with its overflow mode. Loops like `acc += x` keep their type; choose the accumulator format by the type of `acc`. |
| `a * b` | exact, in a format with the word and fraction lengths added (at most 64 bits) |
| two formats with the same modes | promoted to the format that holds both exactly; the result keeps it |
| formats with different modes | no promotion: convert explicitly |
| `a + n`, `a * n`, `a < n` with an integer `n` | exact, in the format of `a` |
| `a + 0.5`, `a < 0.1`, … with a float | `ArgumentError`. Write the constant as `X(0.5)`, which also works for float `X`. |
| `divide(Y, a, b)` | the exact quotient, rounded once to the format `Y`. Division by zero gives what ±Inf converts to, and 0/0 gives 0. |
| `a / b`, `inv(a)` | `divide(typeof(a), a, b)`: the quotient keeps the format of the numerator, as sums keep the format of their operands, and saturates when it does not fit. `n / b` with an integer `n` takes the format of `b`. |
| `a / b` of complex values | `a·conj(b) / abs2(b)` with exact products, one rounding per component, in the format of `a` |
| `sqrt(a)` | the exact root, rounded once, in the format of `a`. Negative values give 0. |
| `exp`, `log`, `sin`, `atan`, `abs(::Complex)`, `angle`, `cis`, `^p`, and complex `sqrt`, `exp`, … | `ArgumentError`. Base would compute them in floating point; a fixed-point algorithm needs its own method. `x^2` works. |
| `ldexp(a, e)` | `a·2^e` in the format of `a`, with its rounding and overflow modes: the scaling step of block floating point |
| `withfraction(a, F)` | the same stored integer with `F` fraction bits: `a·2^(fractionlength(a) - F)`, exact. Followed by a conversion, a right shift with the target's rounding mode. |
| `float(a)`, `Float64(a)` | the exact value when the float has enough significand bits. `float` picks `Float32` up to 24 bits and `Float64` above. |
| `ComplexF64(z)`, `float(z)` of a complex value | per component, as for real values |

## Rounding and overflow modes

Rounding modes are the Base ones: `RoundDown` (floor, an arithmetic right shift), `RoundUp`, `RoundToZero`,
`RoundNearest` (ties to even), `RoundNearestTiesAway` and `RoundNearestTiesUp` (round half up). Overflow modes are
[`Wrap`](@ref), [`Saturate`](@ref) and [`SaturateSymmetric`](@ref).

## Raw values

The stored integer is available with [`rawvalue`](@ref), and [`fromraw`](@ref) builds a value from one, for example
to compare with integer samples from another implementation. Both take complex values too: `rawvalue(z)` is a
complex integer, and `fromraw(Complex{X}, re, im)` reads I/Q pairs. `reinterpret(X, x)` is the same as
`fromraw(X, rawvalue(x))`: like `fromraw`, it brings the integer into the word length of `X` with its overflow mode.

Integer code that shifts after a multiply, such as `(x * y) >> s`, is modeled by computing with fraction length 0
and moving the binary point with [`withfraction`](@ref) before the conversion that rounds:

```julia
I16 = fixed(16, 0; overflow=Wrap)
I32 = fixed(32, 0; overflow=Wrap)                            # srai: round down
N32 = fixed(32, 0; rounding=RoundNearestTiesUp, overflow=Wrap) # add 2^(s-1), then srai

x, y = fromraw(Complex{I16}, 1200, -300), fromraw(Complex{I16}, 950, 410)
p = conj(x) * y                       # exact: madd_epi16, 32-bit sums that wrap
I32(withfraction(real(p), 8))         # one per-component shift, rounded down
Complex{N32}(withfraction(p, 8))      # both components, rounded to nearest
```
