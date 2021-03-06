cmake_minimum_required(VERSION 2.8)

Project(AnyRPC CXX)

# Some of the cmake find_package files are part of this distribution
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# Read out version from "version" file
file(STRINGS "version" ANYRPC_VERSION_FILE)

#parse the version information into pieces.
string(REGEX REPLACE "^v([0-9]+)\\..*" "\\1" ANYRPC_VERSION_MAJOR "${ANYRPC_VERSION_FILE}")
string(REGEX REPLACE "^v[0-9]+\\.([0-9]+).*" "\\1" ANYRPC_VERSION_MINOR "${ANYRPC_VERSION_FILE}")
string(REGEX REPLACE "^v[0-9]+\\.[0-9]+\\.([0-9]+).*" "\\1" ANYRPC_VERSION_PATCH "${ANYRPC_VERSION_FILE}")
string(REGEX REPLACE "^v[0-9]+\\.[0-9]+\\.[0-9]+ +(.*)" "\\1" ANYRPC_VERSION_QUALIFIER "${ANYRPC_VERSION_FILE}")

set(ANYRPC_VERSION ${ANYRPC_VERSION_MAJOR}.${ANYRPC_VERSION_MINOR}.${ANYRPC_VERSION_PATCH})

message(STATUS "AnyRPC Version: ${ANYRPC_VERSION_MAJOR}.${ANYRPC_VERSION_MINOR}.${ANYRPC_VERSION_PATCH} (${ANYRPC_VERSION_QUALIFIER})")
set(ANYRPC_VERSION_STRING "${ANYRPC_VERSION_MAJOR}.${ANYRPC_VERSION_MINOR}.${ANYRPC_VERSION_PATCH} (${ANYRPC_VERSION_QUALIFIER})" )

# File version.h is only regenerated on CMake configure step
CONFIGURE_FILE( "${PROJECT_SOURCE_DIR}/version.h.in"
                "${PROJECT_SOURCE_DIR}/include/anyrpc/version.h" )

# compile in release with debug info mode by default
set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Build Type")

# Build all binaries in a separate directory
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

option(BUILD_EXAMPLES "Build AnyRPC examples." ON)
option(BUILD_TESTS "Build AnyRPC unit tests." OFF)
option(BUILD_WITH_ADDRESS_SANITIZE "Build address sanitizer." OFF)
option(BUILD_WITH_LOG4CPLUS "Build log4cplus." ON)
option(BUILD_WITH_THREADING "Build with threading. Requires c++11 compiler." ON)
option(BUILD_WITH_REGEX "Build with regular expression. Requires c++11 compiler." ON)
option(BUILD_WITH_WCHAR "Build with wide character interface for Value." ON)

option(BUILD_PROTOCOL_JSON "Build with Json protocol included." ON)
option(BUILD_PROTOCOL_XML "Build with Xml procotol included." ON)
option(BUILD_PROTOCOL_MESSAGEPACK "Build iwth MessgePack protocol included." ON)

set(ANYRPC_ASSERT "throw" CACHE STRING "action to take on failed assertion")
set_property(CACHE ANYRPC_ASSERT PROPERTY STRINGS assert throw no_action)

set(ANYRPC_THROW "throw" CACHE STRING "action to take on throw call")
set_property(CACHE ANYRPC_THROW PROPERTY STRINGS assert throw)

#add extra search paths for libraries and includes
set(INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "The directory the headers are installed in")
set(LIB_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE STRING "Directory where lib will install")

include_directories(${CMAKE_SOURCE_DIR}/include)

if (BUILD_PROTOCOL_JSON)
    add_definitions( -DANYRPC_INCLUDE_JSON )
endif ()

if (BUILD_PROTOCOL_XML)
    add_definitions( -DANYRPC_INCLUDE_XML )
endif ()

if (BUILD_PROTOCOL_MESSAGEPACK)
    add_definitions( -DANYRPC_INCLUDE_MESSAGEPACK )
endif ()

if (MSVC)
    add_definitions( -D _CRT_SECURE_NO_WARNINGS )
elseif (MINGW)
    SET( CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -U__STRICT_ANSI__" )
else ()
    SET( CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -pthread" )
    if (BUILD_WITH_ADDRESS_SANITIZE)
        SET( CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer" )
        SET( ASAN_LIBRARY asan )
    endif ()
endif ()

if (BUILD_WITH_THREADING OR BUILD_WITH_REGEX AND NOT MSVC)
    SET( CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -std=c++11" )
endif ()

if (BUILD_WITH_THREADING)
    add_definitions( -DANYRPC_THREADING )
endif ()

if (BUILD_WITH_REGEX)
    add_definitions( -DANYRPC_REGEX )
endif ()

if (BUILD_WITH_WCHAR)
    add_definitions( -DANYRPC_WCHAR )
endif ()

if (ANYRPC_ASSERT STREQUAL "assert")
    add_definitions( -DANYRPC_ASSERT=2 )
elseif (ANYRPC_ASSERT STREQUAL "throw")
    add_definitions( -DANYRPC_ASSERT=1 )
endif ()

if (ANYRPC_THROW STREQUAL "assert")
    add_definitions( -DANYRPC_THROW=2 )
else ()
    add_definitions( -DANYRPC_THROW=1 )
endif ()

find_package( Log4cplus )
find_package( GTest )

add_subdirectory(src)

if (BUILD_EXAMPLES)
    add_subdirectory(example)
endif()

if (BUILD_TESTS)
    add_subdirectory(test)
endif()

install(DIRECTORY include/anyrpc
    DESTINATION "${INCLUDE_INSTALL_DIR}"
    COMPONENT dev)
