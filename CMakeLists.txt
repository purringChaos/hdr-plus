cmake_minimum_required(VERSION 3.10)
project(HDR_PLUS)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

SET(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR})
include("macro.cmake")
set(CMAKE_CXX_STANDARD 11)

# Define dependencies
link_libtiff()

find_package(Halide REQUIRED)
find_package(ZLIB REQUIRED)
find_package(PNG REQUIRED)
find_package(JPEG REQUIRED)
find_library(LIBRAW_LIBRARY NAMES raw raw_r)

set(src_files
    src/InputSource.cpp
    src/Burst.cpp
    src/LibRaw2DngConverter.cpp)

set(header_files
    src/InputSource.h
    src/Burst.h
    src/dngwriter.h
    src/LibRaw2DngConverter.h)

include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${HALIDE_DISTRIB_DIR}/include ${HALIDE_DISTRIB_DIR}/tools ${RAW2DNG_INCLUDE_DIRS})
include_directories(${CMAKE_BINARY_DIR}/genfiles ${CMAKE_BINARY_DIR})

add_executable(hdrplus_pipeline_exe src/hdrplus_pipeline_generator.cpp src/align.cpp src/merge.cpp src/finish.cpp src/util.cpp)
target_link_libraries(hdrplus_pipeline_exe PRIVATE Halide::Generator)

add_halide_library(hdrplus_pipeline FROM hdrplus_pipeline_exe
    GENERATOR hdrplus_pipeline
    FUNCTION_NAME hdrplus_pipeline
)

add_executable(align_and_merge_exe src/align_and_merge_generator.cpp src/align.cpp src/merge.cpp src/util.cpp)
target_link_libraries(align_and_merge_exe PRIVATE Halide::Generator)

add_halide_library(align_and_merge FROM align_and_merge_exe
    GENERATOR align_and_merge
    FUNCTION_NAME align_and_merge
)


add_executable(hdrplus bin/HDRPlus.cpp ${src_files})
add_dependencies(hdrplus hdrplus_pipeline)
target_link_libraries(hdrplus PRIVATE hdrplus_pipeline Halide png ${LIBRAW_LIBRARY} ${TIFF_LIBRARIES} ${TIFFXX_LIBRARY}  Halide::Halide)

add_executable(stack_frames bin/stack_frames.cpp ${src_files})
add_dependencies(stack_frames align_and_merge)
target_link_libraries(stack_frames PRIVATE Halide align_and_merge ${LIBRAW_LIBRARY} ${PNG_LIBRARIES} ${JPEG_LIBRARIES} ${TIFF_LIBRARIES} ${TIFFXX_LIBRARY} Halide::Halide)
