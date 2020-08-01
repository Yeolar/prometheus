# Copyright 2020-present Yeolar

file(GLOB PROME_SRCS *.cc)
file(GLOB PROME_HDRS *.h)

file(GLOB PROME_DETAIL_SRCS detail/*.cc)
file(GLOB PROME_DETAIL_HDRS detail/*.h)

add_library(prometheus OBJECT ${PROME_SRCS} ${PROME_DETAIL_SRCS})

install(FILES ${PROME_HDRS} DESTINATION include/prometheus)
install(FILES ${PROME_DETAIL_HDRS} DESTINATION include/prometheus/detail)
