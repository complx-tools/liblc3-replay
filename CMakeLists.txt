cmake_minimum_required(VERSION 3.5)
# Ignore warnings about rpath behavior on OS X
cmake_policy(SET CMP0042 NEW)
project(liblc3-replay LANGUAGES CXX VERSION 4.20.0)
include(GNUInstallDirs)

# Add code coverage stuff.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

find_package(liblc3 REQUIRED QUIET)
find_package(Boost COMPONENTS serialization iostreams REQUIRED QUIET)
include_directories(${LIBLC3_INCLUDE_DIR})
include_directories(${Boost_INCLUDE_DIRS})

option(ENABLE_TESTS "Enable testing" OFF)
option(ENABLE_COVERAGE "Enable testing coverage" OFF)

if(ENABLE_TESTS)
    enable_testing()
endif(ENABLE_TESTS)

# Require C++11 build
if(NOT CMAKE_CXX_FLAGS MATCHES "-std=(c|gnu)\\+\\+17")
    message(STATUS "This project requires C++17. Adding -std=c++17 to CXXFLAGS.")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17" CACHE STRING "Flags used by the compiler during all build types." FORCE)
endif()

# Additional debug flags
set(CMAKE_CXX_FLAGS_DEBUG "-g -pg")
set(CMAKE_DEBUG_POSTFIX d)

if(ENABLE_COVERAGE)
    include(CodeCoverage)
    append_coverage_compiler_flags()
endif(ENABLE_COVERAGE)


# Set default warning flags
set(PROJECT_WARNING_FLAGS "-Wall -Wno-deprecated -Wno-deprecated-declarations" CACHE STRING "Compiler warning flags to include")
mark_as_advanced(PROJECT_WARNING_FLAGS)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${PROJECT_WARNING_FLAGS}")

# Default to release build if not specified
if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif(NOT CMAKE_BUILD_TYPE)

if(CMAKE_BUILD_TYPE MATCHES "Debug")
    set(CMAKE_EXE_LINKER_FLAGS "-fprofile-arcs -ftest-coverage")
endif(CMAKE_BUILD_TYPE MATCHES "Debug")

add_definitions(-DPREFIX=${CMAKE_INSTALL_PREFIX})
add_definitions(-DSO_SUFFIX="${CMAKE_SHARED_LIBRARY_SUFFIX}")

set(CMAKE_SKIP_RPATH TRUE)

set(
    SRC_FILES
    source/lc3_replay.cpp
    source/BinaryStreamReader.cpp
)

set(
    HEADER_FILES
    include/lc3_replay/lc3_replay.hpp
    include/lc3_replay/BinaryStreamReader.hpp
)
    
add_library(
    lc3_replay
    STATIC
    ${SRC_FILES}
)

# Use target_sources when 3.23 is standard.

target_link_libraries(
    lc3_replay
    ${LIBLC3_LIBRARY}
    ${Boost_LIBRARIES}
)

target_include_directories(
    lc3_replay
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        ${CMAKE_CURRENT_SOURCE_DIR}/include/lc3_replay
    INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

set_property(TARGET lc3_replay PROPERTY POSITION_INDEPENDENT_CODE ON)

export(
    TARGETS lc3_replay
    NAMESPACE liblc3_replay:: 
    FILE ${PROJECT_BINARY_DIR}/cmake/lc3/liblc3_replay-export.cmake
)

install(
    TARGETS lc3_replay
    EXPORT liblc3_replay COMPONENT dev
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime
    # FILE_SET HEADERS DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} COMPONENT dev
)

install(
    EXPORT liblc3_replay
    NAMESPACE   liblc3_replay::
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/liblc3_replay/cmake
    COMPONENT dev
)

install(
    FILES ${HEADER_FILES} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lc3_replay
)

install(
    FILES include/lc3_replay.hpp DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

if(ENABLE_TESTS)
    add_subdirectory(tests)
endif(ENABLE_TESTS)

include(CMakePackageConfigHelpers)

set(INCLUDE_INSTALL_DIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE PATH "include path")

configure_package_config_file(
    liblc3_replayConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/liblc3_replayConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/liblc3_replay/cmake
    PATH_VARS INCLUDE_INSTALL_DIR
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/liblc3_replayConfigVersion.cmake
  COMPATIBILITY ExactVersion
)

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/liblc3_replayConfig.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/liblc3_replayConfigVersion.cmake
    DESTINATION
        ${CMAKE_INSTALL_DATAROOTDIR}/liblc3_replay/cmake
)

# uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)