find_package(Boost COMPONENTS unit_test_framework REQUIRED QUIET)
include_directories(${Boost_INCLUDE_DIRS})
include_directories(../include)

if(CMAKE_HOST_UNIX)
    add_definitions(-DBOOST_TEST_DYN_LINK=1)
endif(CMAKE_HOST_UNIX)

set(SRC_TESTS
    lc3_replay_test.cpp
)

add_executable(
    lc3_replay_test_suite
    ${SRC_TESTS}
)

target_link_libraries(
    lc3_replay_test_suite
    ${LIBLC3_LIBRARY}
    ${Boost_UNIT_TEST_FRAMEWORK_LIBRARY}
)

set_target_properties(lc3_replay_test_suite PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

add_test(NAME lc3_replay_test_suite
         WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
         COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/lc3_replay_test_suite --log_level=message)

if(ENABLE_COVERAGE)
    set(CMAKE_SKIP_RPATH FALSE)
    set(CMAKE_INSTALL_RPATH ${CMAKE_BINARY_DIR})

    set(COVERAGE_EXCLUDES 'boost/*' 'c++/*' 'tests/*')

    setup_target_for_coverage(
        NAME "lc3_test_coverage"
        EXECUTABLE lc3_replay_test_suite -j ${PROCESSOR_COUNT}
        DEPENDENCIES lc3_replay_test_suite)
endif(ENABLE_COVERAGE)
