# Copyright (C) 2020, Yeolar

# Define project
project(prometheus)

cmake_minimum_required(VERSION 3.2)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake-scripts)

# Set project version
set(VERSION_MAJOR 1)
set(VERSION_MINOR 0)
set(VERSION_PATCH 0)

# Required packages
set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
set(Boost_USE_MULTITHREADED TRUE)
find_package(Threads REQUIRED)
find_package(GFlags REQUIRED)

# Optional packages
find_package(GTest)

# Setup environment
set(CMAKE_BUILD_TYPE RelWithDebInfo)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")    # memcheck
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_VERBOSE_MAKEFILE OFF)

# Link libraries
link_libraries(
    ${GFLAGS_LIBRARIES}
    ${CMAKE_THREAD_LIBS_INIT}
    ${CMAKE_DL_LIBS}
)

include_directories(
    ${PROJECT_SOURCE_DIR}
    ${CMAKE_PREFIX_PATH}/include
)

# Check and setup configurations
include(CheckCXXSourceCompiles REQUIRED)
include(CheckFunctionExists REQUIRED)

# Check and setup configurations
if(GFLAGS_FOUND)
    set(CMAKE_REQUIRED_LIBRARIES "gflags")

    check_cxx_source_compiles("
        #include <gflags/gflags.h>
        void foo() { google::GetArgv(); }
        int main() { return 0; }"
        PROME_UNUSUAL_GFLAGS_NAMESPACE)

    if(PROME_UNUSUAL_GFLAGS_NAMESPACE)
        set(PROME_GFLAGS_NAMESPACE "google")
    else()
        set(PROME_GFLAGS_NAMESPACE "gflags")
    endif()
endif()

configure_file(
    ${PROJECT_SOURCE_DIR}/prometheus/prometheus-config.h.in
    ${PROJECT_SOURCE_DIR}/prometheus/prometheus-config.h
)

# Work on all sub directories
add_subdirectory(prometheus)

add_library(prometheus_static STATIC
    $<TARGET_OBJECTS:prometheus>
)
set_target_properties(prometheus_static PROPERTIES OUTPUT_NAME prometheus)

add_library(prometheus_shared SHARED
    $<TARGET_OBJECTS:prometheus>
)
set_target_properties(prometheus_shared PROPERTIES OUTPUT_NAME prometheus)

install(TARGETS prometheus_static
    DESTINATION lib
    PERMISSIONS OWNER_READ OWNER_WRITE
                GROUP_READ
                WORLD_READ
)
install(TARGETS prometheus_shared
    DESTINATION lib
    PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE
                GROUP_READ GROUP_EXECUTE
                WORLD_READ WORLD_EXECUTE
)

# Binary

# Test
if(GTEST_FOUND)
    enable_testing()
    add_subdirectory(prometheus/test)
endif()

# Pack
set(CPACK_SYSTEM_NAME "amd64")
find_program(DPKG_CMD dpkg)
if(DPKG_CMD)
    execute_process(
        COMMAND "${DPKG_CMD}" --print-architecture
        OUTPUT_VARIABLE CPACK_SYSTEM_NAME
        OUTPUT_STRIP_TRAILING_WHITESPACE)
endif()

include(InstallRequiredSystemLibraries)
set(CPACK_GENERATOR "DEB;TXZ")
set(CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT "yeolar@qq.com")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Prometheus C++ library.")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libboost-all-dev,libgflags-dev")
include(CPack)

