# FixedPointArithmetic.jl

[![Stable](https://img.shields.io/badge/docs-stable-blue.svg)](https://JGummessonAtroshi.github.io/FixedPointArithmetic.jl/stable/)
[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://JGummessonAtroshi.github.io/FixedPointArithmetic.jl/dev/)
[![CI](https://github.com/JGummessonAtroshi/FixedPointArithmetic.jl/actions/workflows/CI.yml/badge.svg?branch=main)](https://github.com/JGummessonAtroshi/FixedPointArithmetic.jl/actions/workflows/CI.yml?query=branch%3Amain)
[![Coverage](https://codecov.io/gh/JGummessonAtroshi/FixedPointArithmetic.jl/branch/main/graph/badge.svg)](https://codecov.io/gh/JGummessonAtroshi/FixedPointArithmetic.jl)

Binary fixed-point numbers for simulating fixed-point signal processing in Julia: receivers, decoders, filters. A
format has a word length, a fraction length, a rounding mode and an overflow mode. Algorithms written generically
for floating point run unchanged on it, and arrays of a fixed-point type quantize whatever is stored into them.

```julia
using Pkg
Pkg.add("FixedPointArithmetic")
```

A first example:

```julia
using FixedPointArithmetic

Q15 = fixed(16, 15)          # RoundDown, Saturate by default
a = Q15.(0.3 .* randn(1000)) # quantize: round down, saturate
p = a .* a                   # exact products, Fx{32, 30, …}
y = similar(a); y .= p       # stored back as Q15: rounded once
```

See the [documentation](https://JGummessonAtroshi.github.io/FixedPointArithmetic.jl/dev/) for the arithmetic model, performance
notes and the API reference.

## Use of AI

Substantial parts of the code, tests and documentation were written with Claude Code. All of it was reviewed by the
author.
