cmake_minimum_required ( VERSION 2.0 )

set ( PROJECT_VERSION_MAJOR 4 )
set ( PROJECT_VERSION_MINOR 6 )
project ( json_spirit ) # $PROJECT_NAME

set ( SRC_DIR "${PROJECT_SOURCE_DIR}/src" )

# CMake Environment
option ( VERBOSE "Turn on verbose makefile output" off )
option ( DEBUG "Enable full debugging on built library" off )
option ( BUILD_SHARED_LIBS "Build as dynamic library" on )

if ( CMAKE_SYSTEM_NAME STREQUAL "Darwin" )
  option ( UNIVERSAL "Build as an OSX Universal Library" off )
endif ( CMAKE_SYSTEM_NAME STREQUAL "Darwin" )

if ( VERBOSE )
  set ( CMAKE_VERBOSE_MAKEFILE true )
endif ( VERBOSE )

if ( DEBUG )
  set ( CMAKE_BUILD_TYPE Debug )
else ( NOT DEBUG )
  set ( CMAKE_BUILD_TYPE Release )
endif ( DEBUG )

if ( UNIVERSAL )
  set ( CMAKE_OSX_ARCHITECTURES i386; x86_64 )
  message ( STATUS "Building an OSX Universal Library" )
endif ( UNIVERSAL )

if ( BUILD_SHARED_LIBS )
  message ( STATUS "Building shared library" )
else ( NOT BUILD_SHARED_LIBS )
  message ( STATUS "Building static library" )
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

# Install built library to the standard POSIX locations.
#
# Install location can be changed by passing CMAKE_INSTALL_PREFIX through cmake
# at project build time.

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

# Install my project fork info
install ( FILES "LICENSE.md" "README.md" DESTINATION share/doc/${PROJECT_NAME} )

# Install my cmake module script for finding json_spirit library
install ( DIRECTORY ${PROJECT_SOURCE_DIR}/extra
          DESTINATION share/${PROJECT_NAME}/cmake
          PATTERN ".*" EXCLUDE # do not include those pesky dotfiles
        )

# Uninstall target
configure_file  ( "${PROJECT_SOURCE_DIR}/cmake/uninstall.cmake.in"
                  "${CMAKE_CURRENT_BINARY_DIR}/uninstall.cmake"
                  IMMEDIATE @ONLY
                )

add_custom_target ( uninstall
                    COMMAND ${CMAKE_COMMAND} -P
                    ${CMAKE_CURRENT_BINARY_DIR}/uninstall.cmake
                  )

include ( CPack )
