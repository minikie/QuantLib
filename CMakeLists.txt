cmake_minimum_required(VERSION 3.15.0)

project(QuantLib)

# Default build type for single-config generators (set this before project() command)
# For multi-config generators, such as Visual Studio, use: cmake --build . --config=<CONFIG>
if (NOT CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE STREQUAL "")
    set(CMAKE_BUILD_TYPE "Release" CACHE
        STRING "Build type for single-config generators" FORCE)
endif()

project(${PACKAGE_NAME} LANGUAGES CXX DESCRIPTION "The QuantLib C++ Library")

# Path for package-local cmake modules
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})

# Installation directories
set(QL_INSTALL_BINDIR "bin" CACHE STRING "Installation directory for executables")
set(QL_INSTALL_LIBDIR "lib" CACHE STRING "Installation directory for libraries")
set(QL_INSTALL_INCLUDEDIR "include" CACHE STRING "Installation directory for headers")
set(QL_INSTALL_EXAMPLESDIR "lib/QuantLib/examples" CACHE STRING
    "Installation directory for examples")
set(QL_INSTALL_CMAKEDIR "lib/cmake/${PACKAGE_NAME}" CACHE STRING
    "Installation directory for CMake scripts")

# Options
option(QL_BUILD_BENCHMARK "Build benchmark" ON)
option(QL_BUILD_EXAMPLES "Build examples" ON)
option(QL_BUILD_TEST_SUITE "Build test suite" ON)
option(QL_ENABLE_OPENMP "Detect and use OpenMP" OFF)
option(QL_ENABLE_PARALLEL_UNIT_TEST_RUNNER "Enable the parallel unit test runner" OFF)
option(QL_ENABLE_SESSIONS "Singletons return different instances for different sessions" OFF)
option(QL_ENABLE_THREAD_SAFE_OBSERVER_PATTERN "Enable the thread-safe observer pattern" OFF)
option(QL_ENABLE_TRACING "Tracing messages should be allowed" OFF)
option(QL_ENABLE_DEFAULT_WARNING_LEVEL "Enable the default warning level to pass the ci pipeline" ON)
option(QL_COMPILE_WARNING_AS_ERROR "Specify whether to treat warnings on compile as errors." OFF)
option(QL_ERROR_FUNCTIONS "Error messages should include current function information" OFF)
option(QL_ERROR_LINES "Error messages should include file and line information" OFF)
option(QL_EXTRA_SAFETY_CHECKS "Extra safety checks should be performed" OFF)
option(QL_HIGH_RESOLUTION_DATE "Enable date resolution down to microseconds" OFF)
option(QL_THROW_IN_CYCLES "Throw an exception when a notification loop is detected" OFF)
option(QL_FASTER_LAZY_OBJECTS "Cause lazy objects to forward just the first notification instead of every one" ON)
option(QL_NULL_AS_FUNCTIONS "Enable the implementation of Null as template functions" OFF)
option(QL_INSTALL_BENCHMARK "Install benchmark" ON)
option(QL_INSTALL_EXAMPLES "Install examples" ON)
option(QL_INSTALL_TEST_SUITE "Install test suite" ON)
option(QL_TAGGED_LAYOUT "Library names use layout tags" ${MSVC})
option(QL_USE_CLANG_TIDY "Use clang-tidy when building" OFF)
option(QL_USE_INDEXED_COUPON "Use indexed coupons instead of par coupons" OFF)
option(QL_USE_STD_ANY "Use std::any instead of boost::any" OFF)
option(QL_USE_STD_CLASSES "Enable all QL_USE_STD_ options" OFF)
option(QL_USE_STD_FUNCTION "Use std::function and std::bind instead of Boost ones" ON)
option(QL_USE_STD_OPTIONAL "Use std::optional instead of boost::optional" OFF)
option(QL_USE_STD_SHARED_PTR "Use standard smart pointers instead of Boost ones" OFF)
option(QL_USE_STD_TUPLE "Use std::tuple instead of boost::tuple" ON)
set(QL_EXTERNAL_SUBDIRECTORIES "" CACHE STRING "Optional list of external source directories to be added to the build (semicolon-separated)")
set(QL_EXTRA_LINK_LIBRARIES "" CACHE STRING "Optional extra link libraries to add to QuantLib")

# Require C++14 or higher
if (NOT DEFINED CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 14)
elseif(CMAKE_CXX_STANDARD LESS 14)
    message(FATAL_ERROR "Please specify CMAKE_CXX_STANDARD of 14 or higher")
endif()
if (NOT DEFINED CMAKE_CXX_STANDARD_REQUIRED)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()
# Avoid use of compiler language extensions, i.e. -std=c++14 not -std=gnu++14
if (NOT DEFINED CMAKE_CXX_EXTENSIONS)
    set(CMAKE_CXX_EXTENSIONS FALSE)
endif()

# Convenience option to activate all STD options
if (QL_USE_STD_CLASSES)
    if (CMAKE_CXX_STANDARD GREATER_EQUAL 17)
        set(QL_USE_STD_ANY ON)
        set(QL_USE_STD_OPTIONAL ON)
    endif()
    set(QL_USE_STD_FUNCTION ON)
    set(QL_USE_STD_SHARED_PTR ON)
    set(QL_USE_STD_TUPLE ON)
endif()

if (MSVC)
    set(MSVC_RUNTIME "dynamic" CACHE STRING "MSVC runtime to link")
    set_property(CACHE MSVC_RUNTIME PROPERTY STRINGS static dynamic)
    configure_msvc_runtime()

    if(${MSVC_RUNTIME} STREQUAL "static")
        if(USE_BOOST_DYNAMIC_LIBRARIES)
            message(FATAL_ERROR "Use of shared Boost libraries while compiling with static runtime seems not be a good idea.")
        endif()
        set(Boost_USE_STATIC_RUNTIME ON)
    endif()

    # Remove warnings
    add_definitions(-D_SCL_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_WARNINGS)
endif()

# On single-configuration builds, select a default build type that
# gives the same compilation flags as a default autotools build.
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE "RelWithDebInfo")
endif()

# to reference headers via <ql/foo.hpp>, we need to add the root
# directory of the project to includes
include_directories(${CMAKE_CURRENT_SOURCE_DIR} )

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

find_package(Boost)
if (Boost_FOUND)
  include_directories(${Boost_INCLUDE_DIRS})
endif (Boost_FOUND)

add_subdirectory(ql)
if (QL_BUILD_EXAMPLES)
    add_subdirectory(Examples)
endif()
if (QL_BUILD_TEST_SUITE OR QL_BUILD_BENCHMARK)
    add_subdirectory(test-suite)
endif()

# CPack support (make package, make package_source)
set(CPACK_PACKAGE_VERSION_MAJOR ${QUANTLIB_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${QUANTLIB_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${QUANTLIB_VERSION_PATCH})
set(CPACK_PACKAGE_FILE_NAME ${PACKAGE_NAME}-${PACKAGE_VERSION})
set(CPACK_GENERATOR "TGZ" "ZIP" "7Z")
set(CPACK_SOURCE_PACKAGE_FILE_NAME ${PACKAGE_NAME}-${PACKAGE_VERSION})
set(CPACK_SOURCE_GENERATOR "TGZ" "ZIP" "7Z")
set(CPACK_SOURCE_IGNORE_FILES
    "~$"
    "\\\\.swp$"
    "/[Bb]uild"
    "/\\\\.app"
    "/\\\\.cla"
    "/\\\\.cod"
    "/\\\\.git"
    "/\\\\.lgt"
    "/\\\\.mis")
include(CPack)
