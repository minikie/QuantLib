get_quantlib_library_name(QL_OUTPUT_NAME)

file(GLOB_RECURSE QUANTLIB_FILES "*.hpp" "*.cpp")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ../../lib)

message( STATUS "Host Architecture: ${CMAKE_HOST_SYSTEM_PROCESSOR}")
message( STATUS "Build Architecture: ${CMAKE_OSX_ARCHITECTURES}")
message( STATUS "System Name: ${CMAKE_SYSTEM_NAME}")

if(WIN32)
    if (${BUILD_SHARED_LIBS})
        # Windows needs to link static library (nothing is declared to export in QuantLib)
        message(FATAL_ERROR "You should build QuantLib in STATIC mode as nothing is exported from the library. Use BUILD_SHARED_LIBS=OFF")
    endif()
    add_library(${QL_OUTPUT_NAME} ${QUANTLIB_FILES})
    set_property(TARGET ${QL_OUTPUT_NAME} PROPERTY PROJECT_LABEL "QuantLib")
elseif( ${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    set (CMAKE_CXX_STANDARD 11)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations -Wno-pragma-messages -Wno-user-defined-warnings")
    add_library(${QL_OUTPUT_NAME} STATIC ${QUANTLIB_FILES})
else() # for osx
    set (CMAKE_CXX_STANDARD 11)
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