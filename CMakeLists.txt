# Copyright (C) 2020, Yeolar

cmake_minimum_required(VERSION 3.2)

# Define project
project(prometheus VERSION 0.9.0)

include(GNUInstallDirs)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake-scripts)

# Required packages
set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
find_package(Threads REQUIRED)

# Optional packages
find_package(GTest)
find_package(GMock)

# Setup environment
option(BUILD_SHARED_LIBS "Build libraries as shared ones" OFF)

set(CMAKE_BUILD_TYPE RelWithDebInfo)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")    # memcheck
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_VERBOSE_MAKEFILE OFF)

# Link libraries
link_libraries(
  ${CMAKE_THREAD_LIBS_INIT}
  ${CMAKE_DL_LIBS}
)

include_directories(
  ${PROJECT_SOURCE_DIR}
)

# Check and setup configurations
include(CheckCXXSourceCompiles REQUIRED)
include(CheckFunctionExists REQUIRED)

# Check and setup configurations
configure_file(
  ${PROJECT_SOURCE_DIR}/prometheus/prometheus-config.h.in
  ${PROJECT_SOURCE_DIR}/prometheus/prometheus-config.h
)

# Work on all sub directories
add_subdirectory(prometheus)

add_library(prometheus
  $<TARGET_OBJECTS:prometheus_base>
)

install(TARGETS prometheus
  EXPORT prometheus-targets
  DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

# Binary

# Test
if(GTEST_FOUND)
  enable_testing()
  add_subdirectory(prometheus/test)
endif()

# CmakeConfig
include(CMakePackageConfigHelpers)

install(
  EXPORT prometheus-targets
  NAMESPACE prometheus::
  FILE prometheus-targets.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/prometheus
)

configure_package_config_file(
  ${CMAKE_MODULE_PATH}/prometheus-config.cmake.in
  prometheus-config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/prometheus
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  PATH_VARS CMAKE_INSTALL_INCLUDEDIR
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/prometheus-config.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/prometheus
)

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
if(DPKG_CMD)
  set(CPACK_GENERATOR "DEB;TXZ")
else()
  set(CPACK_GENERATOR "TXZ")
endif()
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT "yeolar@qq.com")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Prometheus C++ library.")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "")
include(CPack)

