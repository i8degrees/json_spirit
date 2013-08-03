cmake_minimum_required ( VERSION 2.0 )

set ( PROJECT_VERSION_MAJOR 4 )
set ( PROJECT_VERSION_MINOR 6 )
set ( PROJECT_VERSION_PATCH 1 )
project ( json_spirit ) # $PROJECT_NAME

# Where to find our CMake templates; must be without closing backslash character
set ( CMAKE_TEMPLATE_DIR "${PROJECT_SOURCE_DIR}/cmake" )

# Source files specified path must be relative path from project root and
# without the closing backslash character
set ( SRC_DIR "json_spirit" )

# Include examples & tests
SUBDIRS ( json_demo
          json_headers_only_demo
          json_map_demo
          json_test
        )

include_directories ( ${SRC_DIR} )

# Default installation paths are relative to CMAKE_INSTALL_PREFIX and must be
# without the closing backslash character.
set ( INSTALL_LIBRARY_PATH "lib" )
set ( INSTALL_HEADERS_PATH "include/${PROJECT_NAME}" )
set ( INSTALL_DOCS_PATH "share/doc/${PROJECT_NAME}" )
set ( INSTALL_EXTRA_FILES_PATH "share/${PROJECT_NAME}/cmake" )

# This auto-generated file saves the result of the CMake options relating to the
# feature set for the convenience of controlling these properties without
# modifying the header file sources directly.
configure_file  (
                  "${CMAKE_TEMPLATE_DIR}/json_spirit_config.h.in"
                  "${PROJECT_SOURCE_DIR}/${SRC_DIR}/json_spirit_config.h"
                )

# CMake Environment
option ( VERBOSE "Turn on verbose makefile output" off )
option ( DEBUG "Enable full debugging on built library" off )
option ( BUILD_SHARED_LIBS "Build as dynamic library" on )

# These options influence build time and intermediate file size
option ( ENABLE_VALUE "Enable Value support" on )
option ( ENABLE_UNICODE "Enable Unicode support" on )
option ( ENABLE_MAP "Enable std::map support" on )
option ( ENABLE_UMAP "Enable std::map Unicode support" on )

if ( CMAKE_SYSTEM_NAME STREQUAL "Darwin" ) # OSX specific options
  option ( UNIVERSAL "Build as an OSX Universal Library" off )
  option ( FRAMEWORK "Build as a Framework" off )
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

if ( FRAMEWORK )
  message ( STATUS "Building Framework library" )
endif ( FRAMEWORK )

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

if ( BUILD_SHARED_LIBS )
  add_library ( ${PROJECT_NAME} SHARED ${JSON_SPIRIT_SRCS} )
else ( NOT BUILD_SHARED_LIBS )
  add_library ( ${PROJECT_NAME} STATIC ${JSON_SPIRIT_SRCS} )
endif ( BUILD_SHARED_LIBS )

if ( CMAKE_SYSTEM_NAME STREQUAL "Darwin" ) # OSX specific options

  # For distribution with user's frameworks / application bundle as a dylib
  # (We follow Apple's suggested Framework hierarchy here).
  #
  # TODO / I should probably be using @rpath here
  set ( INSTALL_NAME_PATH "@executable_path/../Libraries" )

  if ( FRAMEWORK )

    set ( INSTALL_NAME_PATH "@executable_path/../Frameworks" )

    # Set the library property with the variable we set above
    set_target_properties ( ${PROJECT_NAME} PROPERTIES
                            BUILD_WITH_INSTALL_RPATH 1
                            INSTALL_NAME_DIR
                            "${INSTALL_NAME_PATH}"
                        )

    # This builds and "installs" the framework in the current build folder
    # Normally, it isn't wise to be setting this variable... but unless I wish
    # to be burdened with the installation process, I see little other
    # reasonable options here.
    set ( CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_BINARY_DIR}" )

    # Set the path of the framework bundle for helping with the forming of
    # install paths
    set ( FRAMEWORK_PATH "${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}.framework" )

    # Overwrite the installation defaults with the anatomy of an OSX Framework
    set ( INSTALL_LIBRARY_PATH "${CMAKE_INSTALL_PREFIX}" )
    set ( INSTALL_HEADERS_PATH "${FRAMEWORK_PATH}/Headers" )
    set ( INSTALL_DOCS_PATH "${FRAMEWORK_PATH}/Resources" )
    set ( INSTALL_EXTRA_FILES_PATH "${FRAMEWORK_PATH}/Resources" )

    # Fill in the Info.plist template with our project details
    set_target_properties ( ${PROJECT_NAME} PROPERTIES
                            FRAMEWORK
                            TRUE
                            MACOSX_FRAMEWORK_INFO_PLIST
                            "${CMAKE_TEMPLATE_DIR}/Info.plist.in"
                            MACOSX_FRAMEWORK_NAME
                            "${PROJECT_NAME}"
                            MACOSX_FRAMEWORK_BUNDLE_VERSION
                            "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}-${CMAKE_BUILD_TYPE}"
                            MACOSX_FRAMEWORK_SHORT_VERSION_STRING
                            "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
                            MACOSX_FRAMEWORK_IDENTIFIER
                            "org.dev.${PROJECT_NAME}"
                          )

  else ( NOT FRAMEWORK )

  set_target_properties ( ${PROJECT_NAME} PROPERTIES
                          BUILD_WITH_INSTALL_RPATH 1
                          INSTALL_NAME_DIR
                          "${INSTALL_NAME_PATH}"
                          )
  endif ( FRAMEWORK )

endif ( CMAKE_SYSTEM_NAME STREQUAL "Darwin" )

# Set library version info
set_target_properties ( ${PROJECT_NAME} PROPERTIES
                        SOVERSION
                        "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}"
                      )

# Install library, dependent upon the type that it is
if ( FRAMEWORK )
  install ( TARGETS ${PROJECT_NAME} LIBRARY FRAMEWORK DESTINATION "${INSTALL_LIBRARY_PATH}" )
else ( NOT FRAMEWORK )
  if ( BUILD_SHARED_LIBS )
    install ( TARGETS ${PROJECT_NAME} LIBRARY DESTINATION "${INSTALL_LIBRARY_PATH}" )
  else ( NOT BUILD_SHARED_LIBS )
    install ( TARGETS ${PROJECT_NAME} ARCHIVE DESTINATION "${INSTALL_LIBRARY_PATH}" )
  endif ( BUILD_SHARED_LIBS )
endif ( FRAMEWORK )

# Install library include headers
install ( FILES ${JSON_SPIRIT_HEADERS} "${SRC_DIR}/json_spirit_config.h"
          DESTINATION "${INSTALL_HEADERS_PATH}"
        )

# Install my project fork info
install ( FILES "LICENSE.md" "README.md" DESTINATION "${INSTALL_DOCS_PATH}" )

# Install my cmake module script for find_package support
install ( DIRECTORY ${PROJECT_SOURCE_DIR}/extra
          DESTINATION "${INSTALL_EXTRA_FILES_PATH}"
          PATTERN ".*" EXCLUDE # do not include those pesky dotfiles
        )

# Uninstall target
configure_file  ( "${CMAKE_TEMPLATE_DIR}/uninstall.cmake.in"
                  "${CMAKE_CURRENT_BINARY_DIR}/uninstall.cmake"
                  IMMEDIATE @ONLY
                )

add_custom_target ( uninstall
                    COMMAND ${CMAKE_COMMAND} -P
                    ${CMAKE_CURRENT_BINARY_DIR}/uninstall.cmake
                  )

include ( CPack )
