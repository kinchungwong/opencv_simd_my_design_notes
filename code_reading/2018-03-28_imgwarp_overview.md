# 2018-03-28 ImgWarp Overview

2018-03-28_imgwarp_overview.md

Because imgwarp is a huge file, this is only a high-level overview. Deeper understanding will be documented in a different markdown file.

# By line number

## 62 - 121

IPP stuff

## 125 - 299

 * Interpolation table initialization 
 * Possibly static
 * Scalar C++

## 301 - 321

 * Tiny helper 

## 327 - 426

 * template <typename T> remapNearest

## 429

 * RemapNoVec -> return 0;

## 435 - 639

 * ifdef (CV_SSE2)
   * RemapVec_8u 
 * else RemapNoVec endif

## 642 - 850

 * ***Line 691***
 * template <CastOp, VecOp, AT> remapBilinear
   * (const Mat src, Mat dst, const Mat xy, const Max fxy, const void* wtab, int borderType, const Scalar borderValue)
 * CastOp is probably scalar.
 * VecOp functor input arguments:
   * The entire input matrix (as typed Mat)
   * Pointer to first output pixel to generate (```D```)
   * Bilinear gather offsets
     * ```XY``` contains a pair of ```short``` for integer part (floored) of source coordinate. 
     * ```FXY``` contains one ```ushort``` per pixel. It specifies an entry in the interpolation weight table.
     * ```wtab``` is the interpolation weight table.
     * ```X1 - dx``` ... todo
   * It remains to be determined whether this ```VecOp``` might read beyond the end of an array. Currently, it is unclear how ```VecOp``` learns of the width of the destination matrix, which also determines the end of ```XY``` and ```FXY```.

## Lines 853 - 954

 * remapBicubic

## Lines 957 - 1064

 * ```remapLanczos4```

## Lines 1074 - 1322

 * class ```RemapInvoker``` : public ```ParallelLoopBody```
 * Contains significant amount of code for on-the-fly conversion
   from several "remap gather offsets" formats into the format used by the resampler output generation kernel

## Lines 1324 - 1545

 * Begin of ```HAVE_OPENCL```
 * List of functions inside this section
   * ```ocl_remap```
   * ```ocl_linearPolar```
   * ```ocl_logPolar``` near Line 1474

## Lines 1547 - 1633
 
 * Begin of ```HAVE_OPENVX```
 * List of functions inside this section
   * ```openvx_remap```

## Lines 1635 - 1685

 * Begin of ```HAVE_IPP && ! IPP_DISABLE_REMAP```
 * class ```IPPRemapInvoker``` : public ```ParallelLoopBody```

## Lines 1689 - 1848

 * Begin of ```CV_EXPORTS_W``` ```cv::remap(...)```

## Lines 1851 - 2217

 * Begin of ```cv::convertMaps(...)```

## Lines 2220 - 

 * class ```WarpAffineInvoker``` : public ```ParallelLoopBody```
 * Appears to be a tile-based loop executor
 * ```BLOCK_SZ``` is hardcoded 64 
 * ifdef ```CV_TRY_SSE4_1``` and runtime (useSSE4_1)
   * call ```opt_SSE4_1::WarpAffineInvoker_Blockline_SSE41(...)```
 * ifdef ```CV_TRY_AVX2``` and runtime (useAVX2)
   * call ```opt_AVX2::warpAffineBlockline(...);```
