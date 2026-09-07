cmake_minimum_required(VERSION 3.23.0)
project(APD)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(POLICY CMP0146)
    cmake_policy(SET CMP0146 OLD)
endif()
if(POLICY CMP0167)
    cmake_policy(SET CMP0167 OLD)
endif()

# recommand >=4.6.0
find_package(OpenCV REQUIRED core imgproc imgcodecs calib3d)
# opencv_calib3d names Eigen3::Eigen in its link interface in this toolchain's
# OpenCVModules.cmake, so the imported target has to exist before generate.
find_package(Eigen3 REQUIRED)
# recommand >=1.78.0
set(Boost_USE_STATIC_LIBS ON)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
find_package(Boost REQUIRED COMPONENTS filesystem system program_options)
# recommand >=11.8
include(CheckLanguage)
check_language(CUDA)
if (CMAKE_CUDA_COMPILER)
    enable_language(CUDA)
else ()
    message(FATAL_ERROR "-- CUDA NOT FOUND: Fatal error")
endif ()

include_directories(
        ${OpenCV_INCLUDE_DIRS}
        ${Boost_INCLUDE_DIRS})
include_directories(.)

set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -O3 --use_fast_math --maxrregcount=128 --ptxas-options=-v -std=c++17 --compiler-options -Wall")

if (CMAKE_COMPILER_IS_GNUCXX)
    add_definitions(-std=c++17)
    add_definitions(-pthread)
    add_definitions(-Wall)
    add_definitions(-Wextra)
    add_definitions(-pedantic)
    add_definitions(-Wno-unused-function)
    add_definitions(-Wno-switch)
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -ffast-math -march=native")
endif ()

add_executable(
        APD
        main.h
        APD.h
        APD.cpp
        APD.cu
        main.cpp
        ThreadPool.h
)

target_link_libraries(
        APD
        "${Boost_LIBRARIES}"
        "${OpenCV_LIBS}"
)

# FindCUDA used to put these on every translation unit's include path; the CUDA
# language does not, and main.h includes cuda_runtime.h from host sources.
target_include_directories(
        APD PUBLIC
        ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}
)
