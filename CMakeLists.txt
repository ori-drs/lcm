cmake_minimum_required(VERSION 3.1)

project(lcm)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
if(CMAKE_VERSION VERSION_LESS 3.7)
  list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/3.7")
endif()

find_package(PkgConfig REQUIRED)
pkg_check_modules(GLIB REQUIRED glib-2.0)

# Configuration and utility functions
include(lcm-cmake/config.cmake NO_POLICY_SCOPE)
include(lcm-cmake/functions.cmake)
include(lcm-cmake/version.cmake)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

# Set CMAKE_MACOSX_RPATH on macOS to satisfy policy CMP0042.
if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  set(CMAKE_MACOSX_RPATH FALSE)
endif()

# Core modules
add_subdirectory(lcm)
add_subdirectory(lcmgen)
add_subdirectory(lcm-logger)

option(LCM_ENABLE_EXAMPLES "Build test and example programs" ON)
if(LCM_ENABLE_EXAMPLES)
  add_subdirectory(liblcm-test)
endif()

# Python
lcm_option(
  LCM_ENABLE_PYTHON
  "Build Python bindings and utilities"
  PYTHON_FOUND Python)
if(LCM_ENABLE_PYTHON)
  add_subdirectory(lcm-python)
endif()

# Documentation (Main, C/C++, .NET)
add_subdirectory(docs)

# Java
lcm_option(
  LCM_ENABLE_JAVA
  "Build Java bindings and utilities"
  JAVA_FOUND Java 1.6)
if(LCM_ENABLE_JAVA)
  add_subdirectory(lcm-java)
  add_custom_target(lcm-spy DEPENDS lcm-spy-alias)
  add_custom_target(lcm-logplayer-gui DEPENDS lcm-logplayer-gui-alias)
endif()

# Lua
lcm_option(
  LCM_ENABLE_LUA
  "Build Lua bindings and utilities"
  LUA_FOUND Lua)
if(LCM_ENABLE_LUA)
  add_subdirectory(lcm-lua)
endif()

# .NET
# TODO

# Go
lcm_option(
  LCM_ENABLE_GO
  "Build Go utilities, bindings is source distributed"

  # Disable until #294 is resolved
  FALSE Go)
  #  GO_FOUND Go)

option(LCM_ENABLE_TESTS "Build unit tests" ON)
if(LCM_ENABLE_TESTS)
  enable_testing()
  add_subdirectory(test)
endif()

# Install rules
include(lcm-cmake/install.cmake)

option(LCM_INSTALL_M4MACROS "Install autotools support M4 macros" ON)
if(LCM_INSTALL_M4MACROS)
  add_subdirectory(m4macros)
endif()

option(LCM_INSTALL_PKGCONFIG "Install pkg-config files" ON)
if(LCM_INSTALL_PKGCONFIG)
  add_subdirectory(lcm-pkgconfig)
endif()

# Distribution rules
add_custom_target(dist
  COMMAND ${CMAKE_COMMAND}
    -DSOURCE_DIR=${lcm_SOURCE_DIR}
    -DOUTPUT_DIR=${lcm_BINARY_DIR}
    -DVERSION=${LCM_VERSION}
    -P ${lcm_SOURCE_DIR}/lcm-cmake/mkdist.cmake)

# CPack packaging rules
include(lcm-cmake/cpack.cmake)
