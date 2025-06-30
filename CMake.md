# Comprehensive CMake Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Basic Concepts](#basic-concepts)
4. [Your First CMake Project](#your-first-cmake-project)
5. [CMakeLists.txt Structure](#cmakeliststxt-structure)
6. [Variables and Properties](#variables-and-properties)
7. [Targets and Libraries](#targets-and-libraries)
8. [Finding and Using External Libraries](#finding-and-using-external-libraries)
9. [Generator Expressions](#generator-expressions)
10. [Custom Commands and Targets](#custom-commands-and-targets)
11. [Testing with CTest](#testing-with-ctest)
12. [Packaging with CPack](#packaging-with-cpack)
13. [Modern CMake Best Practices](#modern-cmake-best-practices)
14. [Advanced Features](#advanced-features)
15. [Common Patterns and Examples](#common-patterns-and-examples)
16. [Troubleshooting](#troubleshooting)

## Introduction

CMake is a cross-platform build system generator that creates native build files for your platform. It doesn't build your code directly; instead, it generates build files (like Makefiles on Unix or Visual Studio projects on Windows) that are then used by your platform's native build tools.

### Why Use CMake?

- **Cross-platform**: Write once, build anywhere
- **Generator-agnostic**: Supports multiple build systems
- **Powerful dependency management**: Easy library integration
- **Scalable**: Works for small projects to large enterprise applications
- **Industry standard**: Used by major projects like OpenCV, Qt, KDE

## Installation

### Windows
- Download from [cmake.org](https://cmake.org/download/)
- Use package managers: `choco install cmake` or `winget install cmake`

### macOS
- Download from [cmake.org](https://cmake.org/download/)
- Use Homebrew: `brew install cmake`
- Use MacPorts: `sudo port install cmake`

### Linux
```bash
# Ubuntu/Debian
sudo apt install cmake

# CentOS/RHEL/Fedora
sudo yum install cmake
# or
sudo dnf install cmake

# Arch Linux
sudo pacman -S cmake
```

### Verify Installation
```bash
cmake --version
```

## Basic Concepts

### Build Process Overview
1. **Configure**: CMake reads CMakeLists.txt and generates build files
2. **Build**: Native build tool compiles the code
3. **Install**: Copy built files to installation directory
4. **Test**: Run tests (optional)
5. **Package**: Create distributable packages (optional)

### Key Terms
- **Source Directory**: Contains your source code and CMakeLists.txt
- **Binary Directory**: Where build files and compiled binaries are generated
- **Target**: Something to be built (executable, library, or custom target)
- **Generator**: Specifies which build tool to use
- **Cache**: Stores configuration variables between runs

## Your First CMake Project

### Project Structure
```
my_project/
├── CMakeLists.txt
├── src/
│   ├── main.cpp
│   └── hello.cpp
├── include/
│   └── hello.h
└── build/          # Created during build
```

### CMakeLists.txt
```cmake
cmake_minimum_required(VERSION 3.15)
project(MyProject VERSION 1.0.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Add executable
add_executable(my_app
    src/main.cpp
    src/hello.cpp
)

# Add include directory
target_include_directories(my_app PRIVATE include)
```

### Source Files
**include/hello.h**
```cpp
#pragma once
#include <string>

std::string get_hello_message(const std::string& name);
```

**src/hello.cpp**
```cpp
#include "hello.h"

std::string get_hello_message(const std::string& name) {
    return "Hello, " + name + "!";
}
```

**src/main.cpp**
```cpp
#include <iostream>
#include "hello.h"

int main() {
    std::cout << get_hello_message("CMake") << std::endl;
    return 0;
}
```

### Building
```bash
mkdir build
cd build
cmake ..
cmake --build .
# or make (on Unix systems)
```

## CMakeLists.txt Structure

### Essential Commands

#### cmake_minimum_required
```cmake
cmake_minimum_required(VERSION 3.15)
```
Specifies the minimum CMake version required.

#### project
```cmake
project(MyProject 
    VERSION 1.0.0 
    DESCRIPTION "A sample project"
    LANGUAGES CXX C
)
```

#### add_executable
```cmake
add_executable(target_name source1.cpp source2.cpp)
```

#### add_library
```cmake
# Static library
add_library(mylib STATIC lib.cpp)

# Shared library
add_library(mylib SHARED lib.cpp)

# Header-only library (CMake 3.19+)
add_library(mylib INTERFACE)
```

#### target_link_libraries
```cmake
target_link_libraries(my_app PRIVATE mylib)
```

## Variables and Properties

### Built-in Variables
```cmake
# Project information
${PROJECT_NAME}
${PROJECT_VERSION}
${PROJECT_SOURCE_DIR}
${PROJECT_BINARY_DIR}

# System information
${CMAKE_SYSTEM_NAME}      # Linux, Windows, Darwin
${CMAKE_SYSTEM_VERSION}
${CMAKE_HOST_SYSTEM_NAME}

# Compiler information
${CMAKE_CXX_COMPILER}
${CMAKE_CXX_COMPILER_ID}  # GNU, Clang, MSVC
${CMAKE_CXX_COMPILER_VERSION}

# Build configuration
${CMAKE_BUILD_TYPE}       # Debug, Release, RelWithDebInfo, MinSizeRel
${CMAKE_CONFIGURATION_TYPES}
```

### Setting Variables
```cmake
# Simple variable
set(MY_VAR "value")

# List variable
set(SOURCE_FILES main.cpp utils.cpp helper.cpp)

# Cache variable (persists between runs)
set(MY_OPTION ON CACHE BOOL "Enable my feature")

# Environment variable
set(ENV{PATH} "/new/path:$ENV{PATH}")
```

### Variable Scope
```cmake
# Function scope
function(my_function)
    set(LOCAL_VAR "local" PARENT_SCOPE)  # Visible to parent
endfunction()

# Directory scope
set(DIR_VAR "directory")

# Global scope
set_property(GLOBAL PROPERTY GLOBAL_VAR "global")
```

### Properties
```cmake
# Set target properties
set_target_properties(my_target PROPERTIES
    CXX_STANDARD 17
    POSITION_INDEPENDENT_CODE ON
    OUTPUT_NAME "my_renamed_target"
)

# Get property
get_target_property(OUTPUT_NAME my_target OUTPUT_NAME)
```

## Targets and Libraries

### Modern CMake Target-Based Approach

Instead of global variables, use target-specific commands:

```cmake
# OLD WAY (avoid)
include_directories(include)
link_directories(/usr/local/lib)
add_definitions(-DMY_DEFINE)

# MODERN WAY
target_include_directories(my_target PRIVATE include)
target_link_directories(my_target PRIVATE /usr/local/lib)
target_compile_definitions(my_target PRIVATE MY_DEFINE)
```

### Visibility Specifiers

- **PRIVATE**: Only for this target
- **PUBLIC**: For this target and targets that link to it
- **INTERFACE**: Only for targets that link to this target

```cmake
add_library(mylib lib.cpp)

# These are used by mylib and anything linking to mylib
target_include_directories(mylib PUBLIC include)
target_compile_features(mylib PUBLIC cxx_std_17)

# These are only used by mylib internally
target_include_directories(mylib PRIVATE src/internal)
target_compile_definitions(mylib PRIVATE INTERNAL_FLAG)

# These are only used by targets linking to mylib
target_compile_definitions(mylib INTERFACE USING_MYLIB)
```

### Library Types

#### Static Libraries
```cmake
add_library(mystaticlib STATIC 
    src/lib1.cpp 
    src/lib2.cpp
)
target_include_directories(mystaticlib PUBLIC include)
```

#### Shared Libraries
```cmake
add_library(mysharedlib SHARED
    src/lib1.cpp
    src/lib2.cpp
)

# Handle symbol visibility
target_compile_definitions(mysharedlib PRIVATE BUILDING_MYLIB)
set_target_properties(mysharedlib PROPERTIES
    CXX_VISIBILITY_PRESET hidden
    VISIBILITY_INLINES_HIDDEN ON
)
```

#### Header-Only Libraries
```cmake
add_library(myheaderlib INTERFACE)
target_include_directories(myheaderlib INTERFACE include)
target_compile_features(myheaderlib INTERFACE cxx_std_17)
```

#### Object Libraries
```cmake
add_library(myobjectlib OBJECT src/common.cpp)
target_include_directories(myobjectlib PRIVATE include)

# Use in other targets
add_executable(app1 src/main1.cpp $<TARGET_OBJECTS:myobjectlib>)
add_executable(app2 src/main2.cpp $<TARGET_OBJECTS:myobjectlib>)
```

## Finding and Using External Libraries

### find_package
```cmake
# Find required package
find_package(Boost REQUIRED COMPONENTS system filesystem)
target_link_libraries(my_app PRIVATE Boost::system Boost::filesystem)

# Find optional package
find_package(OpenCV QUIET)
if(OpenCV_FOUND)
    target_link_libraries(my_app PRIVATE ${OpenCV_LIBS})
    target_include_directories(my_app PRIVATE ${OpenCV_INCLUDE_DIRS})
endif()

# Modern imported targets (preferred)
find_package(Threads REQUIRED)
target_link_libraries(my_app PRIVATE Threads::Threads)
```

### pkg-config Integration
```cmake
find_package(PkgConfig REQUIRED)
pkg_check_modules(GTK3 REQUIRED gtk+-3.0)

target_link_libraries(my_app PRIVATE ${GTK3_LIBRARIES})
target_include_directories(my_app PRIVATE ${GTK3_INCLUDE_DIRS})
target_compile_options(my_app PRIVATE ${GTK3_CFLAGS_OTHER})
```

### FetchContent (CMake 3.11+)
```cmake
include(FetchContent)

FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG release-1.12.1
)

# Make available
FetchContent_MakeAvailable(googletest)

# Use it
target_link_libraries(my_tests PRIVATE gtest_main)
```

### ExternalProject
```cmake
include(ExternalProject)

ExternalProject_Add(
    external_lib
    URL https://example.com/lib.tar.gz
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/external
    BUILD_BYPRODUCTS ${CMAKE_BINARY_DIR}/external/lib/libexternal.a
)

# Create imported target
add_library(External::lib STATIC IMPORTED)
set_target_properties(External::lib PROPERTIES
    IMPORTED_LOCATION ${CMAKE_BINARY_DIR}/external/lib/libexternal.a
    INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_BINARY_DIR}/external/include
)
add_dependencies(External::lib external_lib)
```

## Generator Expressions

Generator expressions are evaluated during build generation, not during configuration.

### Syntax
```cmake
$<condition:true_value>
$<IF:condition,true_value,false_value>
```

### Common Examples
```cmake
# Different flags for different build types
target_compile_options(my_app PRIVATE
    $<$<CONFIG:Debug>:-g -O0>
    $<$<CONFIG:Release>:-O3 -DNDEBUG>
)

# Platform-specific sources
target_sources(my_app PRIVATE
    src/common.cpp
    $<$<PLATFORM_ID:Windows>:src/windows.cpp>
    $<$<PLATFORM_ID:Linux>:src/linux.cpp>
)

# Compiler-specific flags
target_compile_options(my_app PRIVATE
    $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra>
    $<$<CXX_COMPILER_ID:MSVC>:/W4>
)

# Link different libraries based on configuration
target_link_libraries(my_app PRIVATE
    $<$<CONFIG:Debug>:debug_lib>
    $<$<CONFIG:Release>:optimized_lib>
)
```

### Useful Generator Expressions
```cmake
# Target properties
$<TARGET_PROPERTY:target,PROPERTY>
$<TARGET_FILE:target>              # Full path to target file
$<TARGET_FILE_NAME:target>         # Just the filename
$<TARGET_FILE_DIR:target>          # Directory containing target

# Boolean expressions
$<BOOL:value>
$<AND:conditions>
$<OR:conditions>
$<NOT:condition>

# String operations
$<LOWER_CASE:string>
$<UPPER_CASE:string>
$<JOIN:list,separator>
```

## Custom Commands and Targets

### add_custom_command
```cmake
# Generate a file during build
add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/generated.cpp
    COMMAND ${CMAKE_COMMAND} -E echo "// Generated file" > ${CMAKE_BINARY_DIR}/generated.cpp
    DEPENDS ${CMAKE_SOURCE_DIR}/template.cpp.in
    COMMENT "Generating source file"
)

# Add to target
add_executable(my_app main.cpp ${CMAKE_BINARY_DIR}/generated.cpp)
```

### add_custom_target
```cmake
# Custom target that always runs
add_custom_target(generate_docs
    COMMAND doxygen ${CMAKE_SOURCE_DIR}/Doxyfile
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Generating documentation"
)

# Make it part of default build
add_dependencies(my_app generate_docs)
```

### Code Generation Example
```cmake
# Find Python
find_package(Python3 REQUIRED COMPONENTS Interpreter)

# Generate code
add_custom_command(
    OUTPUT ${CMAKE_BINARY_DIR}/generated_code.cpp
    COMMAND ${Python3_EXECUTABLE} ${CMAKE_SOURCE_DIR}/generate.py
            --output ${CMAKE_BINARY_DIR}/generated_code.cpp
            --input ${CMAKE_SOURCE_DIR}/config.json
    DEPENDS ${CMAKE_SOURCE_DIR}/generate.py ${CMAKE_SOURCE_DIR}/config.json
    COMMENT "Generating code from configuration"
)

add_executable(my_app main.cpp ${CMAKE_BINARY_DIR}/generated_code.cpp)
```

## Testing with CTest

### Enable Testing
```cmake
# In top-level CMakeLists.txt
enable_testing()

# Or include CTest module
include(CTest)
```

### Simple Tests
```cmake
# Add test executable
add_executable(my_tests test_main.cpp test_functions.cpp)
target_link_libraries(my_tests PRIVATE my_library)

# Add test
add_test(NAME unit_tests COMMAND my_tests)

# Set test properties
set_tests_properties(unit_tests PROPERTIES
    TIMEOUT 30
    PASS_REGULAR_EXPRESSION "All tests passed"
)
```

### Google Test Integration
```cmake
find_package(GTest REQUIRED)

add_executable(gtest_example test_main.cpp)
target_link_libraries(gtest_example PRIVATE GTest::gtest_main my_library)

# Discover tests automatically
include(GoogleTest)
gtest_discover_tests(gtest_example)
```

### Running Tests
```bash
# Build and run tests
cmake --build . --target test
# or
ctest

# Verbose output
ctest --verbose

# Run specific test
ctest -R "unit_tests"

# Run tests in parallel
ctest -j 4
```

## Packaging with CPack

### Basic Setup
```cmake
# Set package information
set(CPACK_PACKAGE_NAME "MyApplication")
set(CPACK_PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "My amazing application")
set(CPACK_PACKAGE_VENDOR "My Company")

# Platform-specific generators
if(WIN32)
    set(CPACK_GENERATOR "NSIS;ZIP")
elseif(APPLE)
    set(CPACK_GENERATOR "DragNDrop")
else()
    set(CPACK_GENERATOR "TGZ;DEB")
endif()

include(CPack)
```

### Installation Rules
```cmake
# Install executable
install(TARGETS my_app
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

# Install headers
install(DIRECTORY include/
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
)

# Install documentation
install(FILES README.md LICENSE
    DESTINATION share/doc/${PROJECT_NAME}
)
```

### Debian Package Configuration
```cmake
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name <your.email@example.com>")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6, libstdc++6")
set(CPACK_DEBIAN_PACKAGE_SECTION "devel")
set(CPACK_DEBIAN_PACKAGE_PRIORITY "optional")
```

## Modern CMake Best Practices

### Project Structure
```
project/
├── CMakeLists.txt          # Main CMake file
├── cmake/
│   ├── modules/            # Custom find modules
│   └── scripts/            # Utility scripts
├── src/
│   ├── CMakeLists.txt      # Source subdirectory
│   ├── app/                # Application code
│   └── lib/                # Library code
├── tests/
│   └── CMakeLists.txt      # Test configuration
├── docs/                   # Documentation
└── extern/                 # External dependencies
```

### Modern CMake Principles

1. **Use targets, not variables**
```cmake
# Good
target_include_directories(mylib PUBLIC include)
target_compile_features(mylib PUBLIC cxx_std_17)

# Avoid
set(CMAKE_CXX_STANDARD 17)
include_directories(include)
```

2. **Be explicit about requirements**
```cmake
target_compile_features(mylib PUBLIC cxx_std_17)
target_compile_definitions(mylib PRIVATE HAS_FEATURE=1)
```

3. **Use PRIVATE, PUBLIC, INTERFACE appropriately**
```cmake
# Implementation details
target_include_directories(mylib PRIVATE src/internal)

# Public API
target_include_directories(mylib PUBLIC include)

# Requirements for users
target_compile_features(mylib INTERFACE cxx_std_17)
```

4. **Avoid global commands**
```cmake
# Avoid
link_libraries(somelib)
add_definitions(-DMYMACRO)

# Prefer
target_link_libraries(mytarget PRIVATE somelib)
target_compile_definitions(mytarget PRIVATE MYMACRO)
```

### Configuration Management
```cmake
# Generate config header
configure_file(
    ${CMAKE_SOURCE_DIR}/config.h.in
    ${CMAKE_BINARY_DIR}/include/config.h
)

target_include_directories(mylib PRIVATE ${CMAKE_BINARY_DIR}/include)
```

**config.h.in**
```cpp
#pragma once

#define PROJECT_VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define PROJECT_VERSION_MINOR @PROJECT_VERSION_MINOR@

#cmakedefine HAVE_FEATURE_X
#cmakedefine01 ENABLE_LOGGING
```

## Advanced Features

### Functions and Macros
```cmake
# Function (creates new scope)
function(add_my_executable name)
    set(sources ${ARGN})
    add_executable(${name} ${sources})
    target_compile_features(${name} PRIVATE cxx_std_17)
    target_include_directories(${name} PRIVATE include)
endfunction()

# Macro (no new scope)
macro(set_default_properties target)
    set_target_properties(${target} PROPERTIES
        CXX_STANDARD 17
        CXX_STANDARD_REQUIRED ON
    )
endmacro()

# Usage
add_my_executable(myapp src/main.cpp src/utils.cpp)
set_default_properties(myapp)
```

### Advanced Function Arguments
```cmake
function(create_library)
    set(options STATIC SHARED HEADER_ONLY)
    set(oneValueArgs NAME VERSION)
    set(multiValueArgs SOURCES HEADERS DEPENDENCIES)
    
    cmake_parse_arguments(ARG "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})
    
    if(ARG_STATIC)
        add_library(${ARG_NAME} STATIC ${ARG_SOURCES})
    elseif(ARG_SHARED)
        add_library(${ARG_NAME} SHARED ${ARG_SOURCES})
    elseif(ARG_HEADER_ONLY)
        add_library(${ARG_NAME} INTERFACE)
    endif()
    
    if(ARG_DEPENDENCIES)
        target_link_libraries(${ARG_NAME} PRIVATE ${ARG_DEPENDENCIES})
    endif()
endfunction()

# Usage
create_library(
    NAME mylib
    STATIC
    VERSION 1.0
    SOURCES src/lib.cpp
    HEADERS include/lib.h
    DEPENDENCIES fmt::fmt
)
```

### Toolchain Files
**arm-linux-toolchain.cmake**
```cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

set(CMAKE_C_COMPILER arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER arm-linux-gnueabihf-g++)

set(CMAKE_FIND_ROOT_PATH /usr/arm-linux-gnueabihf)
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

**Usage:**
```bash
cmake -DCMAKE_TOOLCHAIN_FILE=arm-linux-toolchain.cmake ..
```

### Presets (CMake 3.19+)
**CMakePresets.json**
```json
{
    "version": 3,
    "configurePresets": [
        {
            "name": "debug",
            "displayName": "Debug Build",
            "binaryDir": "build/debug",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Debug",
                "CMAKE_EXPORT_COMPILE_COMMANDS": "ON"
            }
        },
        {
            "name": "release",
            "displayName": "Release Build",
            "binaryDir": "build/release",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Release"
            }
        }
    ],
    "buildPresets": [
        {
            "name": "debug-build",
            "configurePreset": "debug"
        }
    ]
}
```

**Usage:**
```bash
cmake --preset debug
cmake --build --preset debug-build
```

## Common Patterns and Examples

### Multi-Configuration Project
```cmake
cmake_minimum_required(VERSION 3.15)
project(MultiConfigProject)

# Global settings
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Add subdirectories
add_subdirectory(src)
add_subdirectory(tests)

# Optional features
option(BUILD_EXAMPLES "Build example programs" ON)
if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

# Installation
include(GNUInstallDirs)
install(TARGETS mylib myapp
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
```

### Header-Only Library with Dependencies
```cmake
# Create interface library
add_library(myheaderlib INTERFACE)

# Add include directory
target_include_directories(myheaderlib INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# Require C++17
target_compile_features(myheaderlib INTERFACE cxx_std_17)

# Add dependencies
find_package(fmt REQUIRED)
target_link_libraries(myheaderlib INTERFACE fmt::fmt)

# Export for use in build tree
export(TARGETS myheaderlib
    FILE ${CMAKE_BINARY_DIR}/myheaderlib-targets.cmake
)

# Installation
install(TARGETS myheaderlib
    EXPORT myheaderlib-targets
    INCLUDES DESTINATION include
)
install(DIRECTORY include/ DESTINATION include)
install(EXPORT myheaderlib-targets
    FILE myheaderlib-targets.cmake
    DESTINATION lib/cmake/myheaderlib
)
```

### Conditional Compilation
```cmake
# Check for compiler features
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-std=c++20" COMPILER_SUPPORTS_CXX20)

if(COMPILER_SUPPORTS_CXX20)
    target_compile_features(mylib PUBLIC cxx_std_20)
else()
    target_compile_features(mylib PUBLIC cxx_std_17)
endif()

# Check for system capabilities
include(CheckIncludeFileCXX)
check_include_file_cxx("filesystem" HAVE_FILESYSTEM)

if(HAVE_FILESYSTEM)
    target_compile_definitions(mylib PRIVATE HAVE_FILESYSTEM)
else()
    find_package(Boost REQUIRED COMPONENTS filesystem)
    target_link_libraries(mylib PRIVATE Boost::filesystem)
endif()
```

## Troubleshooting

### Common Issues and Solutions

#### Build Directory Pollution
**Problem:** CMake files mixed with source files
**Solution:** Always use out-of-source builds
```bash
mkdir build && cd build
cmake ..
```

#### Cache Issues
**Problem:** Old configuration cached
**Solution:** Delete cache or build directory
```bash
rm -rf build/
# or
rm CMakeCache.txt
```

#### Library Not Found
**Problem:** `find_package` fails
**Solution:** Set paths or use different find method
```cmake
# Set paths
set(CMAKE_PREFIX_PATH "/path/to/library")

# Or use pkg-config
find_package(PkgConfig REQUIRED)
pkg_check_modules(MYLIB REQUIRED mylib)
```

#### Verbose Output
```bash
# See actual compiler commands
cmake --build . --verbose
# or
make VERBOSE=1

# See find_package details
cmake .. -DCMAKE_FIND_DEBUG_MODE=ON
```

### Debugging CMake
```cmake
# Print variables
message(STATUS "CMAKE_CXX_COMPILER: ${CMAKE_CXX_COMPILER}")
message(DEBUG "Debug message")
message(WARNING "Warning message")

# Print all variables
get_cmake_property(_variableNames VARIABLES)
foreach(_variableName ${_variableNames})
    message(STATUS "${_variableName}=${${_variableName}}")
endforeach()

# Print target properties
get_target_property(SOURCES my_target SOURCES)
message(STATUS "Target sources: ${SOURCES}")
```

### Performance Tips
- Use `ccache` for faster rebuilds
- Enable parallel builds: `cmake --build . -j$(nproc)`
- Use `ninja` generator for faster builds: `cmake -GNinja ..`
- Avoid expensive operations in configuration step

## Conclusion

This guide covers the essential aspects of CMake from basic usage to advanced features. Remember these key principles:

1. Use modern CMake (3.15+) features and patterns
2. Think in terms of targets, not global settings
3. Be explicit about dependencies and requirements
4. Use appropriate visibility specifiers (PRIVATE/PUBLIC/INTERFACE)
5. Keep your CMake code clean and maintainable

CMake is a powerful tool that, when used correctly, makes building and distributing C++ projects much easier across different platforms and build systems. Practice with small projects first, then gradually incorporate more advanced features as needed.

For the most up-to-date information, always refer to the official CMake documentation at [cmake.org](https://cmake.org/documentation/).
