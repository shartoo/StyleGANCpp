cmake_minimum_required(VERSION 3.6)

project(stylegan)

set(EXECUTABLE_NAME "stylegan")

if(MSVC)
    set(COMMON_OPTIONS /W3 /wd4996 /we4239 /we4002 /we4700 /we4305 /EHsc /MP)
    set(DEBUG_OPTIONS /MTd /Od /Zi)
    set(RELEASE_OPTIONS /MT /Ox /fp:fast /GL /GR-)
elseif(APPLE)
    set(COMMON_OPTIONS -DHAVE_PTHREAD)
    set(DEBUG_OPTIONS -g -Wall)
    set(RELEASE_OPTIONS -Ofast -fPIC -funsafe-math-optimizations -fno-strict-aliasing -fno-rtti -ffast-math -flto -msse2 -msse3 -msse4 -fopenmp)
else()
    set(COMMON_OPTIONS -DHAVE_PTHREAD)
    set(DEBUG_OPTIONS -g -Wall)
    set(RELEASE_OPTIONS -Ofast -fPIC -funsafe-math-optimizations -fno-strict-aliasing -fno-rtti -ffast-math -flto -msse2 -msse3 -msse4 -fopenmp)
endif()

set(DEBUG_OPTIONS ${DEBUG_OPTIONS} ${COMMON_OPTIONS})
set(RELEASE_OPTIONS ${RELEASE_OPTIONS} ${COMMON_OPTIONS})

add_compile_options(
        "$<$<CONFIG:RELEASE>:${RELEASE_OPTIONS}>"
        "$<$<CONFIG:DEBUG>:${DEBUG_OPTIONS}>")

if (CMAKE_VERSION VERSION_LESS "3.1")
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        set (CMAKE_CXX_FLAGS "--std=c++14 ${CMAKE_CXX_FLAGS}")
    endif ()
else ()
    set (CMAKE_CXX_STANDARD 14)
endif ()

include_directories(tensor4/include)
include_directories(tensor4/examples/common)
include_directories(zfp/include)
#include_directories(/opt/intel/mkl/include)
#link_directories(/opt/intel/mkl/lib/intel64)

file(GLOB SOURCES_ZFP zfp/src/*.c)

# Main sources
##############################################################
file(GLOB SOURCES main.cpp StyleGAN.cpp StyleGAN.h numpy-like-randn.h)
add_executable(stylegan ${SOURCES} ${SOURCES_ZFP})

# Compressor sources
##############################################################
file(GLOB SOURCES compressor.cpp)
add_executable(compressor ${SOURCES} ${SOURCES_ZFP})

##############################################################
# Linkage
##############################################################
if(MSVC)
else()
    set(LIBRARIES rt m gomp)# mkl_rt)
endif()

target_link_libraries(stylegan ${LIBRARIES})
target_link_libraries(compressor ${LIBRARIES})
