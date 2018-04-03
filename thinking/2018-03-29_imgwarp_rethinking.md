# 2018-03-29 Rethinking imgwarp

 * Filename: [2018-03-29_imgwarp_rethinking.md]

## Types of kernels in imgwarp interpolation

 * For non-rotating imgwarp
   * Expressed as row filters and column filters
   * Compatible with row-based execution and buffering engine
   * Also compatible with tile-based execution engine, with caveats, and possibly lower performance

 * For arbitrary interpolating imgwarp
   * Nearest interpolation requires look-up of precomputed weighted stencil of size 1x1
   * Bilinear interpolation requires look-up of precomputed weighted stencil of size 2x2
   * Bicubic interpolation requires look-up of precomputed weighted stencil of size 4x4
   * Lanczos4 interpolation requires look-up of precomputed weighted stencil of size 8x8

 * Use of other interpolation weight formulas should be allowed, therefore the difference
   in interpolating kernel implementation is merely the size of the stencil, which can be
   1x1, 2x2, 4x4, or 8x8.

 * Non-square-shaped stencil might be needed to speed up Lanczos4
   * Lanczos4 involve reading 64 input pixels from a square-shaped window.
   * These are not even the "nearest", not in the Euclidean sense.
   * In weighted interpolation, the pixels which play the heaviest influences are those 
     with largest magnitude of weights.
   * We can potentially sort by absolute pixel weight, then select a subset of pixels
     for weighted averaging. (Note: pixel weights are signed, for Bicubic and Lanczos4)
   * An example would be selecting 24, 32, 40, 48, 56 pixels from Lanczos4

## Remark: discrete vs true interpolation

 * The use of precomputed coefficients result in discretization of coordinates.
 * It is different from true interpolation, because true interpolation involves evaluation of polynomial.
   * In other words, evaluating output coordinates at any infinitesimal increment should 
     involve actual evaluation of the polynomial, and the validity of the higher-order derivatives of 
     the result should match that of the order of polynomial.

## Remark: anti-aliasing, or mip-mapping

 * This is involved when the area of an output pixel corresponds to an area on the input image
   that is much larger than a few pixels. 
 * For example, if one output pixel maps to a 10x10 pixel area on the input image (totaling 100 
   pixels or more), it would be advantageous if all those input pixels are averaged together
   when computing the output pixel.
 * This is a topic in computer graphics rendering. 
 * It is applicable to image warping in OpenCV. 
 * The logic is actually in reverse: it is not up to OpenCV to reject the best-practices from
   the discipline of computer graphics.

## Typedefs for each interpolation coefficient pack

### Bicubic discrete, or 4x4

 * Precomputed spatial precision: 1/32th (5 fractional bits) of each coordinate axis (x, y)
 * Weight for each pixel in the 4x4 input window, 32i or 32f
 * Total size = (32 * 32) * (4 * 4) * (4 bytes per value) = 64 KBytes (for either 32i or 32f, not both)

