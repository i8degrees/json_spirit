cmake_minimum_required(VERSION 2.8)
SET(JSON_SPIRIT_SRCS
json_spirit_reader.cpp json_spirit_reader.h
json_spirit_value.cpp json_spirit_value.h
json_spirit_writer.cpp json_spirit_writer.h
json_spirit.h
json_spirit_error_position.h
json_spirit_reader_template.h
json_spirit_stream_reader.h
json_spirit_utils.h
json_spirit_writer_options.h
json_spirit_writer_template.h )

FIND_PACKAGE(Boost 1.34 REQUIRED)
INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIR})

#ADD_LIBRARY ( json_spirit SHARED ${JSON_SPIRIT_SRCS} )
ADD_LIBRARY(json_spirit STATIC ${JSON_SPIRIT_SRCS})

set ( JSON_SPIRIT_HEADERS
      json_spirit.h
      json_spirit_error_position.h
      json_spirit_reader.h
      json_spirit_reader_template.h
      json_spirit_stream_reader.h
      json_spirit_utils.h
      json_spirit_value.h
      json_spirit_writer.h
      json_spirit_writer_options.h
      json_spirit_writer_template.h
)


#   Install library

install ( TARGETS json_spirit
          LIBRARY ARCHIVE DESTINATION lib
        )

#   Install headers

install ( FILES ${JSON_SPIRIT_HEADERS}
          DESTINATION include/json_spirit
        )

#   Uninstall target

configure_file  (
                  "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
                  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
                  IMMEDIATE @ONLY
                )

add_custom_target ( uninstall
                    COMMAND ${CMAKE_COMMAND} -P
                    ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
                  )
