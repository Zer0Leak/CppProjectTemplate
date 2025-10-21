cmake_minimum_required(VERSION 3.22)

# ---------- (optional) force Clang: do this BEFORE project() ----------
# If you prefer setting compilers via VS Code settings or CLI, remove this block.
if(NOT DEFINED CMAKE_C_COMPILER AND NOT DEFINED CMAKE_CXX_COMPILER)
  set(CMAKE_C_COMPILER clang)
  set(CMAKE_CXX_COMPILER clang++)
endif()

# ---------- single place to rename your target ----------
set(TARGET_NAME cpp_template_subproject_target_name)

project(${TARGET_NAME} LANGUAGES CXX)

# Default build type = Debug if not set by user
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build." FORCE)
endif()

set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "RelWithDebInfo" "MinSizeRel")

# Good defaults
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Use target-based features (modern CMake) instead of global C++ standard vars
add_executable(${TARGET_NAME}
  src/main.cpp
  src/second.cpp
)

target_include_directories(${TARGET_NAME} PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/include
)

target_compile_features(${TARGET_NAME} PRIVATE cxx_std_23)

# Eigen (Ubuntu 24.04: Eigen 3.4.x from libeigen3-dev)
find_package(Eigen3 3.4 REQUIRED NO_MODULE)

# xtensor (provides INTERFACE target xtensor)
find_package(xtensor REQUIRED)
find_package(xtensor-blas REQUIRED)

# Let xt::linalg use CBLAS (target-scoped)
target_compile_definitions(${TARGET_NAME} PRIVATE HAVE_CBLAS=1)

# Find BLAS/LAPACK (OpenBLAS, etc.)
find_package(BLAS REQUIRED)
find_package(LAPACK REQUIRED)

# Enable BLAS-backed kernels in Blaze (harmless if you’re not using Blaze yet)
target_compile_definitions(${TARGET_NAME} PRIVATE BLAZE_BLAS_MODE=1)

# C++ formatting library (provides INTERFACE target fmt::fmt)
find_package(fmt REQUIRED)

find_package(OpenMP QUIET)
if(OpenMP_CXX_FOUND)
  target_link_libraries(${TARGET_NAME} PRIVATE OpenMP::OpenMP_CXX)
endif()


# --- Blaze (header-only) via FetchContent ---
include(FetchContent)
FetchContent_Declare(
  blaze
  GIT_REPOSITORY https://bitbucket.org/blaze-lib/blaze.git
  GIT_TAG        v3.8.2
)
FetchContent_MakeAvailable(blaze)

# Create an imported interface target so we can link it
add_library(Blaze::Blaze INTERFACE IMPORTED)
set_target_properties(Blaze::Blaze PROPERTIES
  INTERFACE_INCLUDE_DIRECTORIES "${blaze_SOURCE_DIR}"
)

# BLAS is already enabled for Blaze with BLAZE_BLAS_MODE=1)

# If you installed to a non-standard prefix (e.g. /opt/arrayfire), point CMake at it:
# cmake -S . -B build -DArrayFire_DIR=/opt/arrayfire/share/ArrayFire/cmake
find_package(ArrayFire REQUIRED)

# Ubuntu 24.04 has AF 3.3.x with legacy exports. Map whatever exists to AF_TARGET.

# (If you installed the official ArrayFire, set ArrayFire_DIR accordingly.)
# cmake -DArrayFire_DIR=/opt/arrayfire/share/ArrayFire/cmake ...

# Probe modern targets first, then legacy ones, then variables.
if(TARGET ArrayFire::afcpu)
  set(AF_TARGET ArrayFire::afcpu)
elseif(TARGET afcpu)                           # legacy target name
  set(AF_TARGET afcpu)
elseif(TARGET ArrayFire::af)                   # unified (dispatcher)
  set(AF_TARGET ArrayFire::af)
elseif(TARGET af)                              # legacy unified
  set(AF_TARGET af)
elseif(DEFINED ArrayFire_CPU_LIBRARIES)        # very old: only variables
  add_library(AF_CPU_LEGACY INTERFACE)
  target_link_libraries(AF_CPU_LEGACY INTERFACE ${ArrayFire_CPU_LIBRARIES})
  target_include_directories(AF_CPU_LEGACY INTERFACE ${ArrayFire_INCLUDE_DIRS})
  set(AF_TARGET AF_CPU_LEGACY)
else()
  message(FATAL_ERROR
    "No ArrayFire CPU/unified target found. If you used Ubuntu packages, try:\n"
    "  -DArrayFire_DIR=/usr/lib/x86_64-linux-gnu/cmake/ArrayFire\n"
    "Or install the official ArrayFire (newer) and set ArrayFire_DIR accordingly.")
endif()

find_package(PkgConfig REQUIRED)
pkg_check_modules(dlib REQUIRED dlib-1)

target_link_libraries(${TARGET_NAME} PRIVATE
  Eigen3::Eigen
  xtensor
  xtensor-blas
  fmt::fmt
  BLAS::BLAS
  LAPACK::LAPACK
  Blaze::Blaze
  ${AF_TARGET}
  ${dlib_LIBRARIES}
)

target_include_directories(${TARGET_NAME} PRIVATE
    ${dlib_INCLUDE_DIRS}
)

# Warnings (Clang/GCC)
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
  target_compile_options(${TARGET_NAME} PRIVATE -Wall -Wextra -Wpedantic)
endif()

# Helpful hint:
# cmake -S . -B build -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
# cmake --build build
# For Release
# cmake -S . -B build-release -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_BUILD_TYPE=Release
# cmake --build build-release
