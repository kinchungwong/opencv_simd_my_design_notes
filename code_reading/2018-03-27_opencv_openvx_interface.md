# [2018-03-27 OpenCV - OpenVX interface header](./2018-03-27_opencv_openvx_interface.md)

## Tracking the header includes

 * OpenCV (source)
   * "ivx.hpp" in "3rdparty/openvx/include"
     https://github.com/opencv/opencv/blob/master/3rdparty/openvx/include/ivx.hpp
 * OpenVX (include)
   * "VX/vx.h" in "openvx/include"
     https://github.com/GPUOpen-ProfessionalCompute-Libraries/amdovx-core/blob/develop/openvx/include/VX/vx.h
   * "VX/vx_types.h" in "openvx/include"
     https://github.com/GPUOpen-ProfessionalCompute-Libraries/amdovx-core/blob/develop/openvx/include/VX/vx_types.h

## Original motivation

*From around 2018-03-26*

OpenVX puts ```"vx_uint8"``` and similar *typedef*s in the global namespace. In some sense, it has claimed the ```vx_``` prefix for itself. If there is only one thing to remember from reading this note, this is it.

Is this a concern?
*(Hypothetically speaking. In reality, the problem has been prevented.)*

 * Problem wouldn't happen until all three conditions are satisfied:
   * *In case* OpenCV starts adding 256-bit universal intrinsics using same prefix. (It won't be; a new prefix will be decided on.)
   * Some algorithm implementations in OpenCV start using the newly added 256-bit universal intrinsics *based on the same prefix*. 
   * Those same algorithm implementation source files also happen to include OpenVX headers.

How could we catch this earlier?

 * Does OpenCV's build farm enable ```HAVE_OPENVX``` ? If today's answer is no, this is exactly what we must change, preferably today, at least one platform. We should also have automatic tests for some OpenVX functions invoked from OpenCV.

---
