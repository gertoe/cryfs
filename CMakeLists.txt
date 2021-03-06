cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
cmake_policy(SET CMP0054 NEW)

# TODO Perf test:
#  - try if setting CRYPTOPP_NATIVE_ARCH=ON and adding -march=native to the compile commands for cryfs source files makes a difference
#    -> if yes, offer a cmake option to enable both of these

# Determine OS
exec_program(uname OUTPUT_VARIABLE SYSTEM_NAME)
SET(SYSTEM_NAME "${SYSTEM_NAME}" CACHE INTERNAL "")

project(cryfs)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake-utils)
include(utils)

require_gcc_version(5.0)
require_clang_version(4.0)

# Default value is not to build test cases
option(BUILD_TESTING "build test cases" OFF)
option(CRYFS_UPDATE_CHECKS "let cryfs check for updates and security vulnerabilities" ON)
option(DISABLE_OPENMP "allow building without OpenMP libraries. This will cause performance degradations." OFF)

# The following options are helpful for development and/or CI
option(USE_WERROR "build with -Werror flag")
option(USE_CLANG_TIDY "build with clang-tidy checks enabled" OFF)
option(USE_IWYU "build with iwyu checks enabled" OFF)
option(CLANG_TIDY_WARNINGS_AS_ERRORS "treat clang-tidy warnings as errors" OFF)

if(USE_IWYU)
    # note: for iwyu, we need cmake 3.3
    cmake_minimum_required(VERSION 3.3 FATAL_ERROR)
endif()
if(USE_CLANG_TIDY)
    # note: for clang-tidy, we need cmake 3.6, or (if the return code should be handled correctly, e.g. on CI), we need 3.8.
    cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
endif()


if (MSVC)
    option(DOKAN_PATH "Location of the Dokan library, e.g. C:\\Program Files\\Dokan\\DokanLibrary-1.1.0" "")
endif()

# Default value is to build in release mode but with debug symbols
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE INTERNAL "CMAKE_BUILD_TYPE")
endif(NOT CMAKE_BUILD_TYPE)

# The MSVC version on AppVeyor CI needs this
if(MSVC)
  add_definitions(/bigobj)
endif()

add_subdirectory(vendor EXCLUDE_FROM_ALL)
add_subdirectory(src)
add_subdirectory(doc)
add_subdirectory(test)
add_subdirectory(cpack)

if(SYSTEM_NAME STREQUAL "FreeBSD")
  # use a linker from ports, e.g. lld50 from llvm
  message("use a linker built from ports")

  if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang") # using Clang
    # Ensure that FreeBSD build uses current clang from ports
    # which is required to correctly link the required libraries.
    # Since FreeBSD 11.1 bundles clang 4.0.0, require at least 5.0
    message("you have to use clang built from ports")
    require_clang_version(5.0)
  else()
    message(FATAL_ERROR "clang >= 5.0 from ports required")
  endif()
endif(SYSTEM_NAME STREQUAL "FreeBSD")
