
macro(get_quantlib_library_name QL_OUTPUT_NAME)
message(STATUS "QuantLib library name tokens:")

# MSVC: Give QuantLib built library different names following code in 'ql/autolink.hpp'
if(MSVC)
    
    # - toolset
    # ...taken from FindBoost.cmake
    if (NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19.10)
        set(QL_LIB_TOOLSET "-vc141")
    elseif(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19)
        set(QL_LIB_TOOLSET "-vc140")
    elseif(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 18)
        set(QL_LIB_TOOLSET "-vc120")
    elseif(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 17)
        set(QL_LIB_TOOLSET "-vc110")
    elseif(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 16)
        set(QL_LIB_TOOLSET "-vc100")
    else()
        message(FATAL_ERROR "Compiler below VC++2010 is not supported")
    endif()
    message(STATUS " - Toolset: ${QL_LIB_TOOLSET}")
    
    # - platform
    if("${CMAKE_SIZEOF_VOID_P}" EQUAL "8")
        set(QL_LIB_PLATFORM "-x64")
    endif()
    message(STATUS " - Platform: ${QL_LIB_PLATFORM}")

    # - thread linkage
    set(QL_LIB_THREAD_OPT "-mt")  # _MT is defined for /MD and /MT runtimes (https://docs.microsoft.com/es-es/cpp/build/reference/md-mt-ld-use-run-time-library)
    message(STATUS " - Thread opt: ${QL_LIB_THREAD_OPT}")
    
    # - static/dynamic linkage
    if(${MSVC_RUNTIME} STREQUAL "static")
        set(QL_LIB_RT_OPT "-s")
        set(CMAKE_DEBUG_POSTFIX "gd")
    else()
        set(CMAKE_DEBUG_POSTFIX "-gd")
    endif()
    message(STATUS " - Linkage opt: ${QL_LIB_RT_OPT}")
    
    set(${QL_OUTPUT_NAME} "QuantLib${QL_LIB_TOOLSET}${QL_LIB_PLATFORM}${QL_LIB_THREAD_OPT}${QL_LIB_RT_OPT}")
else()
    set(${QL_OUTPUT_NAME} "QuantLib")
endif()
message(STATUS "QuantLib library name: ${${QL_OUTPUT_NAME}}[${CMAKE_DEBUG_POSTFIX}]")
endmacro(get_quantlib_library_name)



get_quantlib_library_name(QL_OUTPUT_NAME)
# set(QL_OUTPUT_NAME, QuantLib)

file(GLOB_RECURSE QUANTLIB_FILES "*.hpp" "*.cpp")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ../../lib)

message( STATUS "Host Architecture: ${CMAKE_HOST_SYSTEM_PROCESSOR}")
message( STATUS "Build Architecture: ${CMAKE_OSX_ARCHITECTURES}")
message( STATUS "System Name: ${CMAKE_SYSTEM_NAME}")
message( STATUS "QL_OUTPUT_NAME : ${QL_OUTPUT_NAME}")

if(WIN32)
    if (${BUILD_SHARED_LIBS})
        # Windows needs to link static library (nothing is declared to export in QuantLib)
        message(FATAL_ERROR "You should build QuantLib in STATIC mode as nothing is exported from the library. Use BUILD_SHARED_LIBS=OFF")
    endif()
    add_library(${QL_OUTPUT_NAME} ${QUANTLIB_FILES})
    set_property(TARGET ${QL_OUTPUT_NAME} PROPERTY PROJECT_LABEL "QuantLib")
elseif( ${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    set (CMAKE_CXX_STANDARD 14)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations -Wno-pragma-messages -Wno-user-defined-warnings")
    add_library(${QL_OUTPUT_NAME} STATIC ${QUANTLIB_FILES})
else() # for osx
    set (CMAKE_CXX_STANDARD 14)
    set (CMAKE_OSX_DEPLOYMENT_TARGET 11.0)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations -Wno-#pragma-messages -Wno-user-defined-warnings")

    if ( ${CMAKE_OSX_ARCHITECTURES} STREQUAL "arm64" )  # arm64
        set(QL_OUTPUT_NAME "QuantLib_arm64")
        add_library(${QL_OUTPUT_NAME} STATIC ${QUANTLIB_FILES})
    else() # x86_64
        add_library(${QL_OUTPUT_NAME} STATIC ${QUANTLIB_FILES})
    endif()

    # set_target_properties(${QL_OUTPUT_NAME} PROPERTIES PREFIX )
endif()

set(QL_LINK_LIBRARY ${QL_OUTPUT_NAME} PARENT_SCOPE)

install(DIRECTORY . DESTINATION ./bin
        FILES_MATCHING PATTERN "*.hpp" PATTERN "*.h")

install(TARGETS ${QL_OUTPUT_NAME}
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
        )