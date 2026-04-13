# Minimum required CMake version.
cmake_minimum_required(VERSION 3.6)
# Project name and settings.
project(idlib-game-engine CXX)
message("building Idlib: Game Engine")

# Enable/disable compilation and execution of unit tests.
# The value of this option can be set from the command-line by -Didlib-game-engine-with-tests=(ON|OFF).
option(idlib-game-engine-with-tests
       "enable/disable compilation and execution of unit tests. ON enables compilation and execution of unit tests, OFF disables compilation and execution of unit tests. Initial value is ON."
       ON)

# Enable/disable compilation of demos.
# The value of this option can be set from the command-line by -Didlib-game-engine-with-demos=(ON|OFF).
option(idlib-game-engine-with-demos
       "enable/disable compilation of demos. ON enables compilation of demos, OFF disables compilation of demos. Initial value is ON."
       ON)

# Idlib integration:
# if a path to Idlib is specified:
#   if the path is relative
#     make it absolute
#   use the existing parent project setup
# Otherwise:
#   fall back to a sibling ../idlib checkout when available
#   otherwise require -DIDLIB_PATH for standalone builds
if (NOT DEFINED IDLIB_PATH)
 if (EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/../idlib/CMakeLists.txt")
  set(IDLIB_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../idlib")
 else()
  message(FATAL_ERROR
          "IDLIB_PATH is not set. In the Egoboo superproject this is provided "
          "by the root CMake project. For standalone idlib-game-engine builds, "
          "pass -DIDLIB_PATH=/path/to/idlib.")
 endif()
 if (NOT IS_ABSOLUTE "${IDLIB_PATH}")
  get_filename_component(IDLIB_PATH "${IDLIB_PATH}" ABSOLUTE BASE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
 endif()
 set(idlib-with-tests OFF CACHE BOOL "enable/disable compilation of tests. ON enables compilation of tests, OFF disables compilation of tests. Initial value is OFF." FORCE)
 add_subdirectory(${IDLIB_PATH})
endif ()

include(${IDLIB_PATH}/buildsystem/set_project_default_properties.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/buildsystem/download_and_unpack.cmake)

#include_google_test()

# Add subdirectories for the library, the tests, and the demos.
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/library)

# Ensure the "check" target exists.
if (NOT MSVC)
  if (NOT TARGET check)
    add_custom_target(check)
  endif()
endif()

if (idlib-game-engine-with-tests)
  add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/tests)
endif()

if (idlib-game-engine-with-demos)
  add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/demos)
endif()
