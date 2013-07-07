cmake_minimum_required ( VERSION 2.8 )

set ( CMAKE_VERBOSE_MAKEFILE true )
set ( CMAKE_BUILD_TYPE Release )

option ( BUILD_SHARED_LIBS "Build as a shared library -- as opposed to static library" on )

if ( BUILD_SHARED_LIBS )
  message ( "Building as a shared library" )
else ( NOT BUILD_SHARED_LIBS )
  message ( "Building as a static library" )
endif ( BUILD_SHARED_LIBS )

set ( JSON_SPIRIT_SRCS
      json_spirit_reader.cpp
      json_spirit_reader.h
      json_spirit_value.cpp
      json_spirit_value.h
      json_spirit_writer.cpp
      json_spirit_writer.h
      json_spirit.h
      json_spirit_error_position.h
      json_spirit_reader_template.h
      json_spirit_stream_reader.h
      json_spirit_utils.h
      json_spirit_writer_options.h
      json_spirit_writer_template.h
    )

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

find_package ( Boost 1.34 REQUIRED )
include_directories ( ${Boost_INCLUDE_DIR} )

if ( NOT BUILD_SHARED_LIBS )
  add_library ( json_spirit STATIC ${JSON_SPIRIT_SRCS} )
else ( BUILD_SHARED_LIBS )
  add_library ( json_spirit SHARED ${JSON_SPIRIT_SRCS} )
endif ( NOT BUILD_SHARED_LIBS )

if ( NOT TTCARDS_JSON_SPIRIT_COMPILE )

  #     Install library
  if ( BUILD_SHARED_LIBS )
    install ( TARGETS json_spirit LIBRARY DESTINATION lib )
  else ( NOT BUILD_SHARED_LIBS )
    install ( TARGETS json_spirit ARCHIVE DESTINATION lib )
  endif ( BUILD_SHARED_LIBS )

  #   Install headers
  install ( FILES ${JSON_SPIRIT_HEADERS} DESTINATION include/json_spirit )

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

endif ( NOT TTCARDS_JSON_SPIRIT_COMPILE )
