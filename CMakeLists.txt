# ---------------------------------------------------------------------------
#  Copyright (c) 2013 Kyle Lutz <kyle.r.lutz@gmail.com>
# 
#  Distributed under the Boost Software License, Version 1.0
#  See accompanying file LICENSE_1_0.txt or copy at
#  http://www.boost.org/LICENSE_1_0.txt
#
# ---------------------------------------------------------------------------

cmake_minimum_required(VERSION 3.0)

project(BoostCompute VERSION 0.5)

list(APPEND CMAKE_MODULE_PATH "${BoostCompute_SOURCE_DIR}/cmake")

add_library(boost_compute INTERFACE)

# find OpenCL
find_package(OpenCL REQUIRED)

target_include_directories(boost_compute SYSTEM INTERFACE ${OPENCL_INCLUDE_DIRS})
target_link_libraries(boost_compute INTERFACE ${OPENCL_LIBRARIES})

# find Boost
find_package(Boost 1.48 REQUIRED)
target_include_directories(boost_compute SYSTEM INTERFACE ${Boost_INCLUDE_DIRS})

# optional support for c++11
option(BOOST_COMPUTE_USE_CPP11 "Use C++11 features" OFF)
if(NOT MSVC)
  if(${BOOST_COMPUTE_USE_CPP11})
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  endif()
endif()

# optional support for offline-caching
option(BOOST_COMPUTE_USE_OFFLINE_CACHE "Use offline cache for OpenCL program binaries" OFF)
if(${BOOST_COMPUTE_USE_OFFLINE_CACHE})
  target_compile_definitions(boost_compute INTERFACE BOOST_COMPUTE_USE_OFFLINE_CACHE)
endif()

# thread-safety options
option(BOOST_COMPUTE_THREAD_SAFE "Compile with BOOST_COMPUTE_THREAD_SAFE defined" OFF)
if(${BOOST_COMPUTE_THREAD_SAFE})
  target_compile_definitions(boost_compute INTERFACE BOOST_COMPUTE_THREAD_SAFE)
  if(${BOOST_COMPUTE_USE_CPP11})
    if(MSVC)
      if (MSVC_VERSION GREATER 1800)
        target_compile_definitions(boost_compute INTERFACE BOOST_COMPUTE_HAVE_THREAD_LOCAL)
      endif()
    else()
      target_compile_definitions(boost_compute INTERFACE BOOST_COMPUTE_HAVE_THREAD_LOCAL)
    endif()
  endif()
endif()

# optional third-party libraries
option(BOOST_COMPUTE_HAVE_EIGEN "Have Eigen" OFF)
option(BOOST_COMPUTE_HAVE_OPENCV "Have OpenCV" OFF)
option(BOOST_COMPUTE_HAVE_QT "Have Qt" OFF)
option(BOOST_COMPUTE_HAVE_VTK "Have VTK" OFF)
option(BOOST_COMPUTE_HAVE_CUDA "Have CUDA" OFF)
option(BOOST_COMPUTE_HAVE_TBB "Have TBB" OFF)
option(BOOST_COMPUTE_HAVE_BOLT "Have BOLT" OFF)

if(${OpenCL_HEADER_CL_EXT_FOUND})
  target_compile_definitions(boost_compute INTERFACE BOOST_COMPUTE_HAVE_HDR_CL_EXT)
endif()

if(MSVC)
  # optional support for boost dynamic libraries
  option(BOOST_COMPUTE_BOOST_ALL_DYN_LINK "Use boost dynamic link libraries" OFF)
  if(${BOOST_COMPUTE_BOOST_ALL_DYN_LINK})
    target_compile_definitions(boost_compute INTERFACE BOOST_ALL_DYN_LINK)
  endif()
  target_compile_definitions(boost_compute INTERFACE _CRT_SECURE_NO_WARNINGS)
  target_compile_definitions(boost_compute INTERFACE _SCL_SECURE_NO_WARNINGS)
endif()

# compiler options
option(BOOST_COMPUTE_ENABLE_COVERAGE "Enable code coverage generation" OFF)

option(BOOST_COMPUTE_BUILD_TESTS "Build the Boost.Compute tests" OFF)
if(${BOOST_COMPUTE_BUILD_TESTS})
  enable_testing()
  add_subdirectory(test)
endif()

option(BOOST_COMPUTE_BUILD_BENCHMARKS "Build the Boost.Compute benchmarks" OFF)
if(${BOOST_COMPUTE_BUILD_BENCHMARKS})
  add_subdirectory(perf)
endif()

option(BOOST_COMPUTE_BUILD_EXAMPLES "Build the Boost.Compute examples" OFF)
if(${BOOST_COMPUTE_BUILD_EXAMPLES})
  add_subdirectory(example)
endif()

### Install ###
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Note: use 'targets_export_name'
configure_file("cmake/Config.cmake.in" "${project_config}" @ONLY)

install(
    TARGETS boost_compute
    EXPORT "${targets_export_name}"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    DIRECTORY "include/boost"
    DESTINATION "${include_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
