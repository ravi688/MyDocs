# Advanced CMake Usage Guide

## Modern CMake Principles

### Target-Based Design
Modern CMake (3.5+) emphasizes targets over directories and variables:

```cmake
# Create interface library for common settings
add_library(common_settings INTERFACE)
target_compile_features(common_settings INTERFACE cxx_std_17)
target_compile_options(common_settings INTERFACE
    $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra -Wpedantic>
    $<$<CXX_COMPILER_ID:Clang>:-Wall -Wextra -Wpedantic>
    $<$<CXX_COMPILER_ID:MSVC>:/W4>
)

# Link targets to inherit settings
target_link_libraries(my_app PRIVATE common_settings)
```

### Generator Expressions
Conditional compilation based on configuration, platform, or compiler:

```cmake
# Configuration-specific settings
target_compile_definitions(my_target PRIVATE
    $<$<CONFIG:Debug>:DEBUG_BUILD>
    $<$<CONFIG:Release>:NDEBUG>
    $<$<PLATFORM_ID:Windows>:WIN32_LEAN_AND_MEAN>
)

# Compiler-specific flags
target_compile_options(my_target PRIVATE
    $<$<AND:$<CXX_COMPILER_ID:GNU>,$<VERSION_GREATER_EQUAL:$<CXX_COMPILER_VERSION>,9.0>>:-fcoroutines>
)

# Complex conditional logic
target_include_directories(my_target PRIVATE
    $<$<BOOL:${USE_EXTERNAL_LIB}>:${EXTERNAL_LIB_INCLUDE_DIR}>
    $<$<NOT:$<BOOL:${USE_EXTERNAL_LIB}>>:${CMAKE_CURRENT_SOURCE_DIR}/internal>
)
```

## Advanced Target Management

### Custom Target Types
```cmake
# Header-only library
add_library(header_only INTERFACE)
target_include_directories(header_only INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# Object library for shared code
add_library(common_objects OBJECT
    src/common.cpp
    src/utilities.cpp
)
target_include_directories(common_objects PRIVATE include)

# Use object library in multiple targets
add_executable(app1 src/app1.cpp $<TARGET_OBJECTS:common_objects>)
add_executable(app2 src/app2.cpp $<TARGET_OBJECTS:common_objects>)
```

### Target Properties and Propagation
```cmake
# Set properties with different visibility levels
set_target_properties(my_lib PROPERTIES
    CXX_STANDARD 20
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
)

# Property propagation control
target_compile_definitions(my_lib
    PRIVATE INTERNAL_DEFINITION
    PUBLIC API_VERSION=2
    INTERFACE CLIENT_CODE_DEFINITION
)
```

## Custom Functions and Macros

### Advanced Function Patterns
```cmake
# Function with named arguments
function(add_custom_executable)
    set(options STATIC_RUNTIME NO_INSTALL)
    set(oneValueArgs NAME VERSION)
    set(multiValueArgs SOURCES DEPENDENCIES COMPILE_DEFINITIONS)
    
    cmake_parse_arguments(ACE "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})
    
    if(NOT ACE_NAME)
        message(FATAL_ERROR "NAME is required")
    endif()
    
    add_executable(${ACE_NAME} ${ACE_SOURCES})
    
    if(ACE_STATIC_RUNTIME)
        set_property(TARGET ${ACE_NAME} PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
    endif()
    
    if(ACE_DEPENDENCIES)
        target_link_libraries(${ACE_NAME} PRIVATE ${ACE_DEPENDENCIES})
    endif()
    
    if(ACE_COMPILE_DEFINITIONS)
        target_compile_definitions(${ACE_NAME} PRIVATE ${ACE_COMPILE_DEFINITIONS})
    endif()
    
    if(NOT ACE_NO_INSTALL)
        install(TARGETS ${ACE_NAME} DESTINATION bin)
    endif()
endfunction()

# Usage
add_custom_executable(
    NAME my_app
    SOURCES src/main.cpp src/app.cpp
    DEPENDENCIES fmt::fmt spdlog::spdlog
    COMPILE_DEFINITIONS APP_VERSION="${PROJECT_VERSION}"
    STATIC_RUNTIME
)
```

### Macro for Code Generation
```cmake
macro(generate_version_header target)
    set(version_file "${CMAKE_CURRENT_BINARY_DIR}/include/${target}_version.h")
    configure_file(
        "${CMAKE_SOURCE_DIR}/cmake/version.h.in"
        "${version_file}"
        @ONLY
    )
    target_include_directories(${target} PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/include")
    target_sources(${target} PRIVATE "${version_file}")
endmacro()
```

## Package and Dependency Management

### FetchContent for Dependencies
```cmake
include(FetchContent)

# Declare multiple dependencies
FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG v1.14.0
    GIT_SHALLOW TRUE
)

FetchContent_Declare(
    json
    GIT_REPOSITORY https://github.com/nlohmann/json.git
    GIT_TAG v3.11.2
    GIT_SHALLOW TRUE
)

# Make available with custom settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
set(JSON_BuildTests OFF CACHE INTERNAL "")

FetchContent_MakeAvailable(googletest json)

# Use the dependencies
target_link_libraries(my_tests PRIVATE gtest_main nlohmann_json::nlohmann_json)
```

### Custom Find Modules
```cmake
# FindCustomLib.cmake
find_path(CUSTOMLIB_INCLUDE_DIR
    NAMES customlib/customlib.h
    PATHS
        /usr/local/include
        /usr/include
        ${CMAKE_PREFIX_PATH}/include
)

find_library(CUSTOMLIB_LIBRARY
    NAMES customlib
    PATHS
        /usr/local/lib
        /usr/lib
        ${CMAKE_PREFIX_PATH}/lib
)

include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(CustomLib
    REQUIRED_VARS CUSTOMLIB_LIBRARY CUSTOMLIB_INCLUDE_DIR
    VERSION_VAR CUSTOMLIB_VERSION
)

if(CustomLib_FOUND AND NOT TARGET CustomLib::CustomLib)
    add_library(CustomLib::CustomLib UNKNOWN IMPORTED)
    set_target_properties(CustomLib::CustomLib PROPERTIES
        IMPORTED_LOCATION "${CUSTOMLIB_LIBRARY}"
        INTERFACE_INCLUDE_DIRECTORIES "${CUSTOMLIB_INCLUDE_DIR}"
    )
endif()
```

## Cross-Platform Configurations

### Platform-Specific Settings
```cmake
# Platform detection and configuration
if(WIN32)
    set(PLATFORM_SOURCES src/windows/platform.cpp)
    set(PLATFORM_LIBRARIES ws2_32 user32)
elseif(APPLE)
    set(PLATFORM_SOURCES src/macos/platform.cpp)
    find_library(COCOA_LIBRARY Cocoa)
    set(PLATFORM_LIBRARIES ${COCOA_LIBRARY})
elseif(UNIX)
    set(PLATFORM_SOURCES src/linux/platform.cpp)
    set(PLATFORM_LIBRARIES pthread dl)
endif()

target_sources(my_app PRIVATE ${PLATFORM_SOURCES})
target_link_libraries(my_app PRIVATE ${PLATFORM_LIBRARIES})
```

### Toolchain Files
```cmake
# arm-linux-toolchain.cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

set(CMAKE_C_COMPILER arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER arm-linux-gnueabihf-g++)

set(CMAKE_FIND_ROOT_PATH /usr/arm-linux-gnueabihf)
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

# Usage: cmake -DCMAKE_TOOLCHAIN_FILE=arm-linux-toolchain.cmake ..
```

## Testing and Quality Assurance

### Advanced Testing Setup
```cmake
include(CTest)

# Custom test categories
function(add_unit_test test_name)
    add_executable(${test_name} tests/${test_name}.cpp)
    target_link_libraries(${test_name} PRIVATE my_lib gtest_main)
    add_test(NAME ${test_name} COMMAND ${test_name})
    set_tests_properties(${test_name} PROPERTIES LABELS "unit")
endfunction()

function(add_integration_test test_name)
    add_executable(${test_name} tests/integration/${test_name}.cpp)
    target_link_libraries(${test_name} PRIVATE my_lib gtest_main)
    add_test(NAME ${test_name} COMMAND ${test_name})
    set_tests_properties(${test_name} PROPERTIES 
        LABELS "integration"
        TIMEOUT 300
    )
endfunction()

# Code coverage
if(CMAKE_BUILD_TYPE STREQUAL "Coverage")
    target_compile_options(my_lib PRIVATE --coverage)
    target_link_options(my_lib PRIVATE --coverage)
    
    find_program(GCOV_PATH gcov)
    find_program(LCOV_PATH lcov)
    find_program(GENHTML_PATH genhtml)
    
    if(GCOV_PATH AND LCOV_PATH AND GENHTML_PATH)
        add_custom_target(coverage
            COMMAND ${LCOV_PATH} --directory . --capture --output-file coverage.info
            COMMAND ${LCOV_PATH} --remove coverage.info '/usr/*' --output-file coverage.info
            COMMAND ${GENHTML_PATH} coverage.info --output-directory coverage_html
            WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
        )
    endif()
endif()
```

## Installation and Packaging

### Comprehensive Installation
```cmake
# Install configuration
include(GNUInstallDirs)

# Install targets
install(TARGETS my_lib my_app
    EXPORT MyProjectTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Install headers
install(DIRECTORY include/ 
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
)

# Generate and install export file
install(EXPORT MyProjectTargets
    FILE MyProjectTargets.cmake
    NAMESPACE MyProject::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)

# Config file generation
include(CMakePackageConfigHelpers)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/MyProjectConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
)
```

### CPack Configuration
```cmake
# Packaging
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "My awesome project")
set(CPACK_PACKAGE_VENDOR "My Company")
set(CPACK_PACKAGE_CONTACT "contact@company.com")

# Platform-specific packaging
if(WIN32)
    set(CPACK_GENERATOR "NSIS;ZIP")
    set(CPACK_NSIS_MODIFY_PATH ON)
elseif(APPLE)
    set(CPACK_GENERATOR "DragNDrop;TGZ")
else()
    set(CPACK_GENERATOR "DEB;RPM;TGZ")
    set(CPACK_DEBIAN_PACKAGE_MAINTAINER "contact@company.com")
    set(CPACK_RPM_PACKAGE_LICENSE "MIT")
endif()

include(CPack)
```

## Performance and Optimization

### Build Performance
```cmake
# Precompiled headers
target_precompile_headers(my_app PRIVATE
    <iostream>
    <vector>
    <string>
    <memory>
)

# Unity builds for faster compilation
set_target_properties(my_lib PROPERTIES
    UNITY_BUILD ON
    UNITY_BUILD_BATCH_SIZE 16
)

# Parallel compilation
if(CMAKE_GENERATOR MATCHES "Make")
    set(CMAKE_MAKE_PROGRAM "${CMAKE_MAKE_PROGRAM} -j${CMAKE_BUILD_PARALLEL_LEVEL}")
endif()
```

### Link-Time Optimization
```cmake
# Enable LTO for release builds
include(CheckIPOSupported)
check_ipo_supported(RESULT ipo_supported OUTPUT ipo_error)

if(ipo_supported)
    set_target_properties(my_app PROPERTIES
        INTERPROCEDURAL_OPTIMIZATION_RELEASE TRUE
        INTERPROCEDURAL_OPTIMIZATION_RELWITHDEBINFO TRUE
    )
endif()
```

## Best Practices Summary

1. **Use Modern CMake**: Prefer target-based commands over directory-based ones
2. **Generator Expressions**: Leverage conditional compilation and configuration
3. **Interface Libraries**: Create reusable configuration packages
4. **Proper Visibility**: Use PUBLIC/PRIVATE/INTERFACE appropriately
5. **Version Requirements**: Specify minimum CMake versions explicitly
6. **Cross-Platform**: Write portable CMake code using built-in modules
7. **Testing Integration**: Include comprehensive testing setup
8. **Documentation**: Comment complex logic and provide usage examples
9. **Performance**: Use unity builds, PCH, and LTO where appropriate
10. **Packaging**: Provide proper installation and packaging support

This advanced usage enables sophisticated build systems that scale from small projects to large, multi-platform applications with complex dependency management and deployment requirements.
