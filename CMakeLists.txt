# CMake build system for exiv2 library and executables
# Copyright 2015-     Daniel Kaneider  <danielkaneider@users.sf.net>
# Copyright 2012-     Robin Mills      <robin@clanmills.com>
# Copyright 2010-2012 Gilles Caulier   <caulier dot gilles at gmail dot com>
# Copyright 2008      Patrick Spendrin <ps_ml at gmx dot de>

# Redistribution and use is allowed according to the terms of the BSD license.
# For details see the accompanying COPYING-CMAKE-SCRIPTS file.

##
# Running cmake in a tree in which we have run ./configure ; make
# creates include/exiv2/exv_conf.h which conflicts with cmake's own exv_conf.h
# This causes incorrect compilation and linking errors.
#
# It's OK to delete this as it will be recreated on demand by <exiv2-dir>/Makefile
if( EXISTS "${CMAKE_SOURCE_DIR}/include/exiv2/exv_conf.h"  )
    file(REMOVE "${CMAKE_SOURCE_DIR}/include/exiv2/exv_conf.h")
endif()
##

CMAKE_MINIMUM_REQUIRED( VERSION 3.1.0 )
PROJECT( exiv2 )

if( POLICY CMP0042 )
    cmake_policy(SET CMP0042 NEW) # enable MACOSX_RPATH support
else()
    SET(CMAKE_MACOSX_RPATH 1)
endif()

SET( PACKAGE_COPYRIGHT      "Andreas Huggel" )
SET( PACKAGE_BUGREPORT      "ahuggel@gmx.net" )
SET( PACKAGE                "exiv2" )
SET( PACKAGE_NAME           "exiv2" )
SET( PACKAGE_STRING         "exiv2 0.25" )
SET( PACKAGE_TARNAME        "exiv2" )
SET( PACKAGE_VERSION        "0.25" )
SET( PACKAGE_URL            "http://www.exiv2.org" )
SET( GENERIC_LIB_VERSION    "14.0.0" )
SET( GENERIC_LIB_SOVERSION  "14" )

# options and their default values
OPTION( EXIV2_ENABLE_SHARED        "Build exiv2 as a shared library (dll)"                 ON  )
OPTION( EXIV2_ENABLE_XMP           "Build with XMP metadata support"                       ON  )
OPTION( EXIV2_ENABLE_LIBXMP        "Build a static convenience Library for XMP"            ON  )
OPTION( EXIV2_ENABLE_VIDEO         "Build with video support"                              OFF )
OPTION( EXIV2_ENABLE_PNG           "Build with png support (requires libz)"                ON  )
OPTION( EXIV2_ENABLE_NLS           "Build native language support (requires gettext)"      ON  )
OPTION( EXIV2_ENABLE_PRINTUCS2     "Build with Printucs2"                                  ON  )
OPTION( EXIV2_ENABLE_LENSDATA      "Build including lens data"                             ON  )
OPTION( EXIV2_ENABLE_COMMERCIAL    "Build with the EXV_COMMERCIAL_VERSION symbol set"      OFF )
OPTION( EXIV2_ENABLE_BUILD_SAMPLES "Build the unit tests"                                  ON  )
OPTION( EXIV2_ENABLE_BUILD_PO      "Build translations files"                              OFF )
OPTION( EXIV2_ENABLE_VIDEO         "Build video support into library"                      OFF )
OPTION( EXIV2_ENABLE_WEBREADY      "Build webready support into library"                   OFF )
IF (WIN32)
    OPTION( EXIV2_ENABLE_WIN_UNICODE   "Use Unicode paths (wstring) on Windows"            OFF )
    OPTION( EXIV2_ENABLE_CURL          "USE Libcurl for HttpIo"                            OFF )
    OPTION( EXIV2_ENABLE_SSH           "USE Libssh for SshIo"                              OFF )
ELSE()
    OPTION( EXIV2_ENABLE_CURL          "USE Libcurl for HttpIo"                            ON  )
    OPTION( EXIV2_ENABLE_SSH           "USE Libssh for SshIo"                              ON  )

ENDIF()

# set include path for FindXXX.cmake files
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/config/")

IF( MINGW OR UNIX )
    IF ( CMAKE_CXX_COMPILER STREQUAL "g++" OR CMAKE_C_COMPILER STREQUAL "gcc" )
        ADD_DEFINITIONS(-Wall
                    -Wcast-align
                    -Wpointer-arith
                    -Wformat-security
                    -Wmissing-format-attribute
                    -Woverloaded-virtual
                    -W
                   )
    ENDIF()
ENDIF( MINGW OR UNIX )

IF( EXIV2_ENABLE_COMMERCIAL )
    SET (EXIV2_ENABLE_LENSDATA OFF)
    SET (EXIV2_ENABLE_NLS OFF)
ENDIF()

set (PRIVATE_VAR)
if(${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION} GREATER 2.8)
    set (PRIVATE_VAR PRIVATE)
endif()

# binary as first, since it workarounds old VS solution compatibility problems
INCLUDE_DIRECTORIES( ${CMAKE_SOURCE_DIR}/src/ ${CMAKE_SOURCE_DIR}/include/  ${CMAKE_SOURCE_DIR}/include/exiv2/)

if( MSVC )
    # cmake_policy(SET CMP0008)
    ADD_DEFINITIONS(-DPSAPI_VERSION=1) # to be compatible with <= WinVista (#905)
endif( MSVC )

IF( EXIV2_ENABLE_XMP )
    SET( HAVE_XMP_TOOLKIT ON )
    SET( XMPLIB "do" )
    set(THREADS_PREFER_PTHREAD_FLAG ON)
    find_package(Threads REQUIRED)
ENDIF( EXIV2_ENABLE_XMP )

INCLUDE( config/CMakeChecks.txt )

SET( INSTALL_TARGET_STANDARD_ARGS
     RUNTIME DESTINATION bin
     LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
     ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

##
# add sources from XMPSDK directory
# this needs to be done here to be visible to xmpsdk, src and sample sub directories
SET( XMPSRC ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/ExpatAdapter.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/MD5.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/ParseRDF.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/UnicodeConversions.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/WXMPIterator.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/WXMPMeta.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/WXMPUtils.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/XML_Node.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/XMPCore_Impl.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/XMPIterator.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/XMPMeta-GetSet.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/XMPMeta-Parse.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/XMPMeta-Serialize.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/XMPMeta.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/XMPUtils-FileInfo.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/xmpsdk/src/XMPUtils.cpp
)

# the following lines need to be added to circumvent the error in xmpsdk\src\MD5.h
IF( ${EXV_HAVE_STDINT_H} )
    ADD_DEFINITIONS( -DEXV_HAVE_STDINT_H )
ENDIF( ${EXV_HAVE_STDINT_H} )

SET(SVN_REVISION "0")
IF(EXISTS "${CMAKE_SOURCE_DIR}/.svn")
    FIND_PACKAGE(Subversion QUIET)
    IF(SUBVERSION_FOUND)
        Subversion_WC_INFO(${PROJECT_SOURCE_DIR} svnExiv2)
        SET(SVN_REVISION "${svnExiv2_WC_REVISION}")
    ENDIF()
ENDIF()

CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/src/svn_version.h.in
                ${CMAKE_CURRENT_SOURCE_DIR}/src/svn_version.h @ONLY)
MESSAGE(STATUS  "SVN version:  ${SVN_REVISION}")

IF( NOT MSVC )
  # Issue #722: out of source builds compiled against standard include files such as /usr/local/lib/include/exiv2
  # do not use CREATE_SYMLINK or CMAKE_CAN_SYMLINK as they don't work on CYGWIN
  EXECUTE_PROCESS( WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}           COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/src exiv2)

  IF( EXIV2_ENABLE_BUILD_SAMPLES )
    EXECUTE_PROCESS( WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/samples COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/src exiv2)
  ENDIF( EXIV2_ENABLE_BUILD_SAMPLES )

  IF( EXIV2_ENABLE_BUILD_PO )
    EXECUTE_PROCESS( WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/po      COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/src exiv2)
  ENDIF( EXIV2_ENABLE_BUILD_PO )
ENDIF()

ADD_SUBDIRECTORY( xmpsdk )
ADD_SUBDIRECTORY( src )

IF( EXIV2_ENABLE_BUILD_SAMPLES )
    ADD_SUBDIRECTORY( samples )
ENDIF( EXIV2_ENABLE_BUILD_SAMPLES )

IF( EXIV2_ENABLE_BUILD_PO )
    ADD_SUBDIRECTORY( po )
ENDIF( EXIV2_ENABLE_BUILD_PO )

##
# tests
ADD_CUSTOM_TARGET(tests  COMMAND make test   "EXIV2_BINDIR=${CMAKE_BINARY_DIR}/bin"  WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/test/" )
ADD_CUSTOM_TARGET(teste  COMMAND make teste  "EXIV2_BINDIR=${CMAKE_BINARY_DIR}/bin"  WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/test/" )
ADD_CUSTOM_TARGET(testv  COMMAND make testv  "EXIV2_BINDIR=${CMAKE_BINARY_DIR}/bin"  WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/test/" )
ADD_CUSTOM_TARGET(testvw COMMAND make testvw "EXIV2_BINDIR=${CMAKE_BINARY_DIR}/bin"  WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/test/" )
ADD_CUSTOM_TARGET(testx  COMMAND make testx  "EXIV2_BINDIR=${CMAKE_BINARY_DIR}/bin"  WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/test/" )

##
# http://dev.exiv2.org/boards/3/topics/1364
# effectively does a make doc on the root directory
# has to run 'make config' and './configure'
# and copy bin/taglist to <exiv2dir>/bin/taglist for use by 'make doc'
IF( MINGW OR UNIX OR APPLE)
  ADD_CUSTOM_TARGET(doc
    WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/doc"
    COMMAND chmod +x ./cmake_doc.sh
    COMMAND ./cmake_doc.sh "${CMAKE_BINARY_DIR}"
  )
ENDIF()

# That's all Folks!
##
