# Copyright 2020-present Yeolar

file(GLOB PROME_BASE_SRCS *.cc)
file(GLOB PROME_BASE_HDRS *.h)

file(GLOB PROME_BASE_DETAIL_SRCS detail/*.cc)
file(GLOB PROME_BASE_DETAIL_HDRS detail/*.h)

add_library(prometheus_base OBJECT ${PROME_BASE_SRCS} ${PROME_BASE_DETAIL_SRCS})

install(FILES ${PROME_BASE_HDRS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/prometheus)
install(FILES ${PROME_BASE_DETAIL_HDRS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/prometheus/detail)
