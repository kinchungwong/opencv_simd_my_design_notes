# [2018-04-03 Universal intrinsics - Setting focus and avoiding duplicated effort](./2018-04-03_universal_intrinsics_focus_and_avoid_duplicated_effort.md)

## Load-store

OpenCV's universal intrinsics already provides load-store instructions.

The only addition would be the return-type dispatching based on user-defined implicit 
conversion operator.

## Operations on concatenated vectors

Concatenated vectors refer to sequences of elements that span **more than one** SIMD registers.
For example, a sequence of 32 bytes, or 8 lanes of 32-bit integers, are considered concatenated vectors.

Thanks to a C++ compiler optimization feature called "scalar replacement of arrays" (SRA, or SRoA),
we can easily define structs that wrap a small fixed-size array of SIMD registers, and be assured
that the C++ compiler can optimize away the struct. By "optimizing away", we refer to the C++ compiler
having the freedom of allocating any part of this struct between SIMD register and memory, not 
necessarily contiguous in memory, and the freedom to rearrange or optimize away CPU instructions
without scheduling barriers.

Note that the SRoA optimization requires the struct to never leave the scope of a function. Typically, 
its lifetime must be local (based on a function's scope), and any functions that access the struct 
must have been fully inlined into the same function.

Code example:

```
struct v_int8x32
{
    union
    {
        __m128i m_v128i[2u];
        int8_t m_i8[32u];
    };
};
```

## Extract subsequence from concatenation

"Align" refers to extracting a subsequence (vector) of values from a sequence formed from concatenating two vectors.

Obviously "align" isn't the best keyword to describe this. I'm using this keyword just because SSSE3 has this "PALIGNR" instruction.

Example:

 * Input vector one is { A, B, C, D }
 * Input vector two is { E, F, G, H }
 * Possible outputs from "align":
   * { A, B, C, D } (offset zero)
   * { B, C, D, E } (offset one)
   * { C, D, E, F } (offset two)
   * { D, E, F, G } (offset three)
   * { E, F, G, H } (offset four)

Remarks:

 * Starting with SSSE3, the instruction PALIGNR allows byte-level extraction 
   from two concatenated vectors using one instruction.
 * The SSSE3 implementation only supports hard-coded (compile-time-constant) offset.
   * This is sufficient for a lot of image convolution tasks, because the convolution 
    window is typically a small fixed size, such as 3x3 or 5x5.
 * Without SSSE3, fallback with SSE2 is available.
 * The SSE2 fallback supports both hard-coded (compile-time-constant) and runtime (variable) offset.

## Lane shift across concatenation

Unlike the subsequence extraction idea, this lane shift idea would return an output that has the 
same number of lanes as the concatenated input. In other words, given an input that spans two
vectors, the output from lane-shift will also span two vectors.

Example:

 * Input vector one is { A, B, C, D }
 * Input vector two is { E, F, G, H }
 * Lane-shift considers the input to be its concatenation, { A, B, C, D, E, F, G, H }
 * Lane-shfit can take a signed offset parameter. With this example of 8 lanes,
   the signed offset can range from minus 8 to plus 8.
 * Example output when signed offset is minus 3: { D, E, F, G, H, ?, ?, ? }
 * Example output when signed offset is plus 7: { ?, ?, ?, ?, ?, ?, ?, A }
 * Example output when signed offset is plus 8: { ?, ?, ?, ?, ?, ?, ?, ? }
   * In this case, the output will not contain any value from the input.
 * The shifted-in fill value might be zero or unspecified, depending on what is more efficient.



