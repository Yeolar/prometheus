# Copyright 2020-present Yeolar

set(PROME_TEST_SRCS
    builder_test.cc
    check_names_test.cc
    counter_test.cc
    family_test.cc
    gauge_test.cc
    histogram_test.cc
    registry_test.cc
    serializer_test.cc
    summary_test.cc
    text_serializer_test.cc
    utils_test.cc
)

foreach(test_src ${PROME_TEST_SRCS})
    get_filename_component(test_name ${test_src} NAME_WE)
    set(test prometheus_${test_name})
    add_executable(${test} ${test_src})
    target_link_libraries(${test} ${GTEST_BOTH_LIBRARIES} prometheus_static)
    add_test(${test} ${test} CONFIGURATIONS ${CMAKE_BUILD_TYPE})
endforeach()

