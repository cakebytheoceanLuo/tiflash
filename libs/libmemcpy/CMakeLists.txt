# Copyright 2022 PingCAP, Ltd.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

option(TIFLASH_FOLLY_MEMCPY_IS_MEMCPY "use folly memcpy as default `memcpy` and `memmove`" ON)

set (memcpy_sources)

# only enbale folly memcpy under linux x86_64 with avx2 supported
if (ARCH_LINUX AND TIFLASH_ENABLE_AVX_SUPPORT)
    set_property(SOURCE folly/memcpy.S PROPERTY LANGUAGE CXX)
    set_property(SOURCE folly/memcpy.S APPEND PROPERTY COMPILE_OPTIONS "-x" "assembler-with-cpp")
    set_property(SOURCE folly/memcpy.S APPEND PROPERTY COMPILE_FLAGS "-mavx -mavx2")
    list (APPEND memcpy_sources folly/memcpy.S)
    message (STATUS "`libmemcpy` support Folly memcpy")
else ()
    set (TIFLASH_FOLLY_MEMCPY_IS_MEMCPY OFF)
endif ()

if (USE_INTERNAL_MEMCPY)
    if (TIFLASH_FOLLY_MEMCPY_IS_MEMCPY)
        message (STATUS "Using Folly memcpy as default `memcpy` and `memmove`")
        add_definitions(-DFOLLY_MEMCPY_IS_MEMCPY=1)
    else ()
        message (STATUS "Using internal memcpy")
        list (APPEND memcpy_sources memcpy.cpp)
    endif ()
else ()
    add_definitions(-DNO_TIFLASH_INTERNAL_MEMCPY=1)
    list (APPEND memcpy_sources memcpy.cpp)
endif()

add_library (memcpy STATIC ${memcpy_sources})
target_include_directories(memcpy PUBLIC ${TiFlash_SOURCE_DIR}/libs/libcommon/include)

set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fomit-frame-pointer")
set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fomit-frame-pointer")
