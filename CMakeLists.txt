cmake_minimum_required(VERSION 3.15.0)

project(QuantLib)

# include(${CMAKE_CURRENT_LIST_DIR}/cmake/quantlib.cmake)

option(BUILD_SHARED_LIBS "Build shared libraries" ${UNIX})
option(USE_BOOST_DYNAMIC_LIBRARIES "Use the shared version of Boost libraries" ${UNIX})
if (USE_BOOST_DYNAMIC_LIBRARIES)
    add_definitions(-DBOOST_ALL_DYN_LINK)
    message( STATUS "Boost Dynamic Libraries is used")
else()
    set(Boost_USE_STATIC_LIBS ON)
    message( STATUS "Boost Static Libraries is used")
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

#
# Copy across the ANSI config file into the build directory. Users
# need to change userconfig.hpp if they require different settings.
#
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/ql/config.ansi.hpp
  ${PROJECT_BINARY_DIR}/config.hpp COPYONLY)
install(FILES ${PROJECT_BINARY_DIR}/config.hpp DESTINATION include/ql)
