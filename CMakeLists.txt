# Copyright (c) Microsoft Corporation
#
# All rights reserved.
#
# MIT License
#
# Permission is hereby granted, free of charge, to any person obtaining a copy of
# this software and associated documentation files (the "Software"), to deal in
# the Software without restriction, including without limitation the rights to
# use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
# of the Software, and to permit persons to whom the Software is furnished to do
# so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED *AS IS*, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required (VERSION 3.10)
project (vlfeat)

include(GNUInstallDirs)
set(INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})
set(INSTALL_CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

# make sure that the default is a RELEASE
set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

if(ENABLE_OPENMP)
  find_package(OpenMP REQUIRED)
endif()

if(USE_SSE)
  set(SSE2_DEFINITIONS -D__SSE2__)
  set(SSE2_VL_C_FILES "vl/mathop_sse2.c" "vl/imopv_sse2.c")
  set(SSE2_VL_H_FILES "vl/mathop_sse2.h" "vl/imopv_sse2.h")
else()
  set(SSE2_DEFINITIONS -DVL_DISABLE_SSE2)
endif()

if(USE_AVX)
  set(AVX_VL_C_FILES "vl/mathop_avx.c")
  set(AVX_VL_H_FILES "vl/mathop_avx.h")
else()
  set(AVX_DEFINITIONS -DVL_DISABLE_AVX)
endif()


set (C_SOURCES
  vl/aib.c
  vl/array.c
  vl/covdet.c
  vl/dsift.c
  vl/fisher.c
  vl/generic.c
  vl/getopt_long.c
  vl/gmm.c
  vl/hikmeans.c
  vl/hog.c
  vl/homkermap.c
  vl/host.c
  vl/ikmeans.c
  vl/imopv.c
  vl/kdtree.c
  vl/kmeans.c
  vl/lbp.c
  vl/liop.c
  vl/mathop.c
  ${AVX_VL_C_FILES}
  ${SSE2_VL_C_FILES}
  vl/mser.c
  vl/pgm.c
  vl/quickshift.c
  vl/random.c
  vl/rodrigues.c
  vl/scalespace.c
  vl/sift.c
  vl/slic.c
  vl/stringop.c
  vl/svm.c
  vl/svmdataset.c
  vl/vlad.c
)

set (H_SOURCES
  vl/aib.h
  vl/array.h
  vl/covdet.h
  vl/dsift.h
  vl/fisher.h
  vl/generic.h
  vl/getopt_long.h
  vl/gmm.h
  vl/heap-def.h
  vl/hikmeans.h
  vl/hog.h
  vl/homkermap.h
  vl/host.h
  vl/ikmeans.h
  vl/imopv.h
  vl/kdtree.h
  vl/kmeans.h
  vl/lbp.h
  vl/liop.h
  vl/mathop.h
  ${AVX_VL_H_FILES}
  ${SSE2_VL_H_FILES}
  vl/mser.h
  vl/pgm.h
  vl/qsort-def.h
  vl/quickshift.h
  vl/random.h
  vl/rodrigues.h
  vl/scalespace.h
  vl/shuffle-def.h
  vl/sift.h
  vl/slic.h
  vl/stringop.h
  vl/svm.h
  vl/svmdataset.h
  vl/vlad.h
)

add_library(vl ${C_SOURCES} ${H_SOURCES})
set_property(TARGET vl PROPERTY POSITION_INDEPENDENT_CODE ON)
target_include_directories(vl PUBLIC $<INSTALL_INTERFACE:${INSTALL_INCLUDE_DIR}> $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/vl>)

if(MSVC)
    target_compile_options(vl PRIVATE
        -D_CRT_SECURE_NO_DEPRECATE
        -D__LITTLE_ENDIAN__
        /Zp8
        /wd4146
    )
endif()

if(CMAKE_COMPILER_IS_GNUCC)
    target_compile_options(vl PRIVATE
        -std=c99
        -Wno-unused-function
        -Wno-long-long
        -Wno-variadic-macros
    )
endif()

target_compile_options(vl PRIVATE ${OpenMP_C_FLAGS})

target_compile_definitions(vl PRIVATE
    -DVL_BUILD_DLL
    ${SSE2_DEFINITIONS}
    ${AVX_DEFINITIONS}
)

set_target_properties(vl PROPERTIES PUBLIC_HEADER "${H_SOURCES}")

install(TARGETS vl EXPORT vlfeatTargets
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  PUBLIC_HEADER DESTINATION "${INSTALL_INCLUDE_DIR}"
  COMPONENT dev
)

install(EXPORT vlfeatTargets
  FILE vlfeatConfig.cmake
  NAMESPACE vlfeat::
  DESTINATION "${INSTALL_CMAKE_DIR}"
)
