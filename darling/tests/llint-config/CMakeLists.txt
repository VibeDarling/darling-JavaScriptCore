cmake_minimum_required(VERSION 3.13)
project(LLIntConfigurationCheck NONE)

# Run cmake -S darling/tests/llint-config -B /tmp/llint-config-check.
# This checks the real selection expression without requiring Darling headers.
get_filename_component(candidate "${CMAKE_CURRENT_LIST_DIR}/../../.." ABSOLUTE)
file(STRINGS "${candidate}/CMakeLists.txt" selection
     REGEX "^[ \t]*set\\(LOW_LEVEL_INTERPRETER_ARM64_CONFIG ")
list(LENGTH selection selection_count)
if(NOT selection_count EQUAL 1)
    message(FATAL_ERROR "Expected one ARM64 interpreter configuration expression")
endif()
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/selection.cmake" "${selection}\n")
include("${CMAKE_CURRENT_BINARY_DIR}/selection.cmake")

if(CMAKE_CONFIGURATION_TYPES)
    set(configurations ${CMAKE_CONFIGURATION_TYPES})
else()
    set(configurations "${CMAKE_BUILD_TYPE}")
endif()
file(GENERATE OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/selected-$<CONFIG>.txt"
     CONTENT "${LOW_LEVEL_INTERPRETER_ARM64_CONFIG}\n")
enable_testing()
# A single-config build can have an empty configuration name.
if(NOT configurations)
    set(configurations "empty")
endif()
foreach(config IN LISTS configurations)
    set(expected debug)
    if(config MATCHES "^(Release|RelWithDebInfo|MinSizeRel)$")
        set(expected release)
    endif()
    set(filename "${config}")
    if(config STREQUAL "empty")
        set(filename "")
    endif()
    add_test(NAME "selection-${config}"
        COMMAND "${CMAKE_COMMAND}"
        "-DINPUT=${CMAKE_CURRENT_BINARY_DIR}/selected-${filename}.txt"
        "-DEXPECTED=${expected}" -P "${CMAKE_CURRENT_LIST_DIR}/check.cmake")
endforeach()
