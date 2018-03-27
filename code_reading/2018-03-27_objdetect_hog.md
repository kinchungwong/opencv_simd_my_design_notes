# 2018-03-27 Code reading - HOG in Object Detection module

HOG stands for *histogram of oriented gradients*.

## Page navigation

 * Current folder: [./README.md](./README.md)
 * Parent folder: [../README.md](../README.md)
 * See LICENSE at the top of directory.

## Original motivation 

*From around 2018-03-26*

 * opencv\modules\objdetect\src\hog.cpp

I was scanning for OpenCV source code which are not using universal intrinsics (in particular certain multiplication instructions), and this is one of the source files I found.

I found that it contains an emulation of ```_mm_mullo_epi32```. Somewhat interestingly, the emulation was used to perform an multiplication with integer value 3, where the input is loaded from memory and output is written back to the same memory. (Both unaligned.)

Since it is not necessary to use actual multiplication instructions when the multiplier is a small integer (3 in this case), I submitted a PR simplifying that. However, looking at the whole source file carefully, I find many other things worth working on. Perhaps this source file will be an interesting motivating case for OpenCV's SIMD improvement.

---

Regarding the emulation of  ```_mm_mullo_epi32```, a suitable implementation already exists in universal intrinsics.

 * opencv/modules/core/include/opencv2/core/hal/intrin_sse.hpp, 
 * ```inline v_uint32x4 operator * (const v_uint32x4& a, const v_uint32x4& b)```

Vector constant splats (setting all lanes of a vector to the same value) is an interesting use case, and there might be unexploited optimization opportunities.

---
