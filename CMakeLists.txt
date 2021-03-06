# Copyright 2011 Free Software Foundation, Inc.
# 
# This file is part of GNU Radio
# 
# GNU Radio is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3, or (at your option)
# any later version.
# 
# GNU Radio is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with GNU Radio; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.


########################################################################
# Project setup
########################################################################
cmake_minimum_required(VERSION 2.6)
project(vrt-dissector C)
set(vrt-dissector_VERSION_MAJOR 0)
set(vrt-dissector_VERSION_MINOR 0)
enable_testing()

#select the release build type by default to get optimization flags
if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE "Release")
   message(STATUS "Build type not specified: defaulting to release.")
endif(NOT CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "")

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)

########################################################################
# Compiler specific setup
########################################################################
#if(CMAKE_COMPILER_IS_GNUC AND NOT WIN32)
#    #http://gcc.gnu.org/wiki/Visibility
#    add_definitions(-fvisibility=visible)
#endif()

find_package(GLIB2)
if(NOT GLIB2_FOUND)
    message(FATAL_ERROR "Glib not found")
endif()

find_package(Wireshark)
if(NOT WIRESHARK_FOUND)
    message(FATAL_ERROR "Wireshark dev package not found")
endif()

########################################################################
# Find boost
########################################################################
if(UNIX AND EXISTS "/usr/lib64")
    list(APPEND BOOST_LIBRARYDIR "/usr/lib64") #fedora 64-bit fix
endif(UNIX AND EXISTS "/usr/lib64")
set(Boost_ADDITIONAL_VERSIONS
    "1.35.0" "1.35" "1.36.0" "1.36" "1.37.0" "1.37" "1.38.0" "1.38" "1.39.0" "1.39"
    "1.40.0" "1.40" "1.41.0" "1.41" "1.42.0" "1.42" "1.43.0" "1.43" "1.44.0" "1.44"
    "1.45.0" "1.45" "1.46.0" "1.46" "1.47.0" "1.47" "1.48.0" "1.48" "1.49.0" "1.49"
    "1.50.0" "1.50" "1.51.0" "1.51" "1.52.0" "1.52" "1.53.0" "1.53" "1.54.0" "1.54"
    "1.55.0" "1.55" "1.56.0" "1.56" "1.57.0" "1.57" "1.58.0" "1.58" "1.59.0" "1.59"
    "1.60.0" "1.60" "1.61.0" "1.61" "1.62.0" "1.62" "1.63.0" "1.63" "1.64.0" "1.64"
    "1.65.0" "1.65" "1.66.0" "1.66" "1.67.0" "1.67" "1.68.0" "1.68" "1.69.0" "1.69"
)
find_package(Boost "1.35")

if(NOT Boost_FOUND)
    message(FATAL_ERROR "Boost required to compile vrt-dissector")
endif()

########################################################################
# Setup the include and linker paths
########################################################################
include_directories(
    ${CMAKE_SOURCE_DIR}
#    ${Boost_INCLUDE_DIRS}
    ${GLIB2_INCLUDE_DIRS}
    ${WIRESHARK_INCLUDE_DIRS}
)

#link_directories(
#    ${Boost_LIBRARY_DIRS}
#)
set(CMAKE_INSTALL_LIBDIR ~/.wireshark)

# Set component parameters
set(VRT_DISSECTOR_INCLUDE_DIRS ${CMAKE_SOURCE_DIR} CACHE INTERNAL "" FORCE)

set(VRT_PLUGIN_SRC ${CMAKE_SOURCE_DIR}/packet-vrt.c)

set(PLUGIN_C_GENERATOR ${CMAKE_SOURCE_DIR}/make-dissector-reg.py)
set(PLUGIN_C plugin.c)

add_custom_command(
        OUTPUT ${PLUGIN_C}
        DEPENDS ${VRT_PLUGIN_SRC}
        COMMAND ${PLUGIN_C_GENERATOR} ${CMAKE_SOURCE_DIR} plugin ${VRT_PLUGIN_SRC}
        COMMENT "Generating ${PLUGIN_C}"
)

add_library(vrt-plugin MODULE
    ${PLUGIN_C}
    moduleinfo.h
    ${VRT_PLUGIN_SRC}
)
set_target_properties(vrt-plugin PROPERTIES PREFIX "")
set_target_properties(vrt-plugin PROPERTIES LINK_FLAGS "${WS_LINK_FLAGS}")
target_link_libraries(vrt-plugin wireshark)

install(TARGETS vrt-plugin
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/plugins NAMELINK_SKIP
)
