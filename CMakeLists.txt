# CMake build system for exiv2 library and executables
# Copyright 2015-     Daniel Kaneider  <danielkaneider@users.sf.net>
# Copyright 2010-2012 Gilles Caulier   <caulier dot gilles at gmail dot com>
# Copyright 2008      Patrick Spendrin <ps_ml at gmx dot de>

# Redistribution and use is allowed according to the terms of the BSD license.
# For details see the accompanying COPYING-CMAKE-SCRIPTS file.

cmake_minimum_required( VERSION 3.1.0 )
project( exiv2 )

include( GNUInstallDirs )

if (NOT MSVC)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
endif()

set(CMAKE_MACOSX_RPATH 1)

set( PACKAGE_COPYRIGHT      "Andreas Huggel" )
set( PACKAGE_BUGREPORT      "http://github.com/exiv2/exiv2" )
set( PACKAGE                "exiv2" )
set( PACKAGE_NAME           "exiv2" )
set( PACKAGE_VERSION        "0.26" )
set( PACKAGE_URL            "http://www.exiv2.org" )
set( PACKAGE_STRING         "${PACKAGE_NAME} ${PACKAGE_VERSION}" )
set( GENERIC_LIB_VERSION    "26.0.0" )
set( GENERIC_LIB_SOVERSION  "26" )

# options and their default values
option( EXIV2_ENABLE_SHARED        "Build exiv2 as a shared library (dll)"                 ON  )
option( EXIV2_ENABLE_XMP           "Build with XMP metadata support"                       ON  )
option( EXIV2_ENABLE_LIBXMP        "Build a static convenience Library for XMP"            ON  )
option( EXIV2_ENABLE_PNG           "Build with png support (requires libz)"                ON  )
option( EXIV2_ENABLE_NLS           "Build native language support (requires gettext)"      ON  )
option( EXIV2_ENABLE_PRINTUCS2     "Build with Printucs2"                                  ON  )
option( EXIV2_ENABLE_LENSDATA      "Build including lens data"                             ON  )
option( EXIV2_ENABLE_COMMERCIAL    "Build with the EXV_COMMERCIAL_VERSION symbol set"      OFF )
option( EXIV2_ENABLE_BUILD_SAMPLES "Build the unit tests"                                  ON  )
option( EXIV2_ENABLE_BUILD_PO      "Build translations files"                              OFF )
option( EXIV2_ENABLE_VIDEO         "Build video support into library"                      OFF )
option( EXIV2_ENABLE_WEBREADY      "Build webready support into library"                   OFF )
option( EXIV2_ENABLE_DYNAMIC_RUNTIME  "Use dynamic runtime (used for static libs)"         OFF )
if (WIN32)
    option( EXIV2_ENABLE_WIN_UNICODE   "Use Unicode paths (wstring) on Windows"            OFF )
    option( EXIV2_ENABLE_CURL          "USE Libcurl for HttpIo"                            OFF )
    option( EXIV2_ENABLE_SSH           "USE Libssh for SshIo"                              OFF )
else()
    option( EXIV2_ENABLE_CURL          "USE Libcurl for HttpIo"                            ON  )
    option( EXIV2_ENABLE_SSH           "USE Libssh for SshIo"                              ON  )
endif()
option( EXIV2_ENABLE_TOOLS         "Build exiv2 executable"                                ON  )

include(config/findDependencies.cmake)
include(config/compilerFlags.cmake)

if( EXIV2_ENABLE_COMMERCIAL )
    set (EXIV2_ENABLE_LENSDATA OFF)
    set (EXIV2_ENABLE_NLS OFF)
endif()

if( MSVC )
    set(CMAKE_DEBUG_POSTFIX "d")
endif()

if( EXIV2_ENABLE_XMP )
    set( HAVE_XMP_TOOLKIT ON )
endif()

include( config/generateConfigFile.cmake )
include_directories(${CMAKE_BINARY_DIR})  # Make the exv_conf.h file visible for the full project

include( config/CMakeChecks.txt )

if( EXIV2_ENABLE_XMP )
    add_subdirectory( xmpsdk )
endif()
add_subdirectory( src )

if( EXIV2_ENABLE_BUILD_SAMPLES )
    add_subdirectory( samples )
endif()

if( EXIV2_ENABLE_BUILD_PO )
    add_subdirectory( po )
endif()

##
# tests
add_custom_target(tests       COMMAND env EXIV2_BINDIR="${CMAKE_BINARY_DIR}"/bin make tests       WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}" )
add_custom_target(geotag-test COMMAND env EXIV2_BINDIR="${CMAKE_BINARY_DIR}"/bin make geotag-test WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}" )

##
# http://dev.exiv2.org/boards/3/topics/1364
# effectively does a make doc on the root directory
# has to run 'make config' and './configure'
# and copy bin/taglist to <exiv2dir>/bin/taglist for use by 'make doc'
if( MINGW OR UNIX OR APPLE)
  add_custom_target(doc
    WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/doc"
    COMMAND chmod +x ./cmake_doc.sh
    COMMAND ./cmake_doc.sh "${CMAKE_BINARY_DIR}"
  )
endif()

include( config/printSummary.cmake )
