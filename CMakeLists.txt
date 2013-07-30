cmake_minimum_required ( VERSION 2.8 )

set ( PROJECT_VERSION_MAJOR 4 )
set ( PROJECT_VERSION_MINOR 6 )
project ( json_spirit ) # $PROJECT_NAME

set ( SRC_DIR "${PROJECT_SOURCE_DIR}/src" )

# CMake Environment
set ( CMAKE_VERBOSE_MAKEFILE false )
set ( CMAKE_BUILD_TYPE Release )

option ( BUILD_SHARED_LIBS "Build as a dylib (dynamic library)" on )
option ( BUILD_UNIVERSAL_LIB "Build as an OSX Universal Library" off )

if ( BUILD_UNIVERSAL_LIB )
  set ( CMAKE_OSX_ARCHITECTURES i386; x86_64 )
  message ( STATUS "Building an OSX Universal Library" )
endif ( BUILD_UNIVERSAL_LIB )

if ( BUILD_SHARED_LIBS )
  message ( STATUS "Building as a shared library" )
else ( NOT BUILD_SHARED_LIBS )
  message ( STATUS "Building as a static library" )
endif ( BUILD_SHARED_LIBS )

set ( JSON_SPIRIT_SRCS
      ${SRC_DIR}/json_spirit_reader.cpp
      ${SRC_DIR}/json_spirit_reader.h
      ${SRC_DIR}/json_spirit_value.cpp
      ${SRC_DIR}/json_spirit_value.h
      ${SRC_DIR}/json_spirit_writer.cpp
      ${SRC_DIR}/json_spirit_writer.h
      ${SRC_DIR}/json_spirit.h
      ${SRC_DIR}/json_spirit_error_position.h
      ${SRC_DIR}/json_spirit_reader_template.h
      ${SRC_DIR}/json_spirit_stream_reader.h
      ${SRC_DIR}/json_spirit_utils.h
      ${SRC_DIR}/json_spirit_writer_options.h
      ${SRC_DIR}/json_spirit_writer_template.h
    )

set ( JSON_SPIRIT_HEADERS
      ${SRC_DIR}/json_spirit.h
      ${SRC_DIR}/json_spirit_error_position.h
      ${SRC_DIR}/json_spirit_reader.h
      ${SRC_DIR}/json_spirit_reader_template.h
      ${SRC_DIR}/json_spirit_stream_reader.h
      ${SRC_DIR}/json_spirit_utils.h
      ${SRC_DIR}/json_spirit_value.h
      ${SRC_DIR}/json_spirit_writer.h
      ${SRC_DIR}/json_spirit_writer_options.h
      ${SRC_DIR}/json_spirit_writer_template.h
    )

# Required Dependencies
find_package ( Boost 1.34 REQUIRED )
include_directories ( ${Boost_INCLUDE_DIR} )

# Install output library in the standard POSIX locations.
#
# CMAKE_INSTALL_PREFIX can be passed to cmake to change this.

if ( BUILD_SHARED_LIBS )

  add_library ( ${PROJECT_NAME} SHARED ${JSON_SPIRIT_SRCS} )
  install ( TARGETS ${PROJECT_NAME} LIBRARY DESTINATION lib )

else ( NOT BUILD_SHARED_LIBS )

  add_library ( ${PROJECT_NAME} STATIC ${JSON_SPIRIT_SRCS} )
  install ( TARGETS ${PROJECT_NAME} ARCHIVE DESTINATION lib )

endif ( BUILD_SHARED_LIBS )

# Set library version info
set_target_properties ( ${PROJECT_NAME} PROPERTIES
                        SOVERSION
                        "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}"
                      )

# Install library include headers
install ( FILES ${JSON_SPIRIT_HEADERS} DESTINATION include/${PROJECT_NAME} )

# Uninstall target
configure_file  (
                  "${PROJECT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
                  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
                  IMMEDIATE @ONLY
                )

add_custom_target ( uninstall
                    COMMAND ${CMAKE_COMMAND} -P
                    ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
                  )

#include ( CPACK )
