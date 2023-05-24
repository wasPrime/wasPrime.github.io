---
title: Configure Cmake
date: 2023-03-31 00:00:00
categories:
- [configuration, cmake]
tags:
- configuration
- cmake
- make
- cmakelists
---

{% note primary %}
A basic `CmakeLists.txt` example from [a WebServer library](https://github.com/wasPrime/WebServer).
Help to quickly start a project.
{% endnote %}

## Basic file tree

```log
.
├── CMakeLists.txt
├── src
│   ├── CMakeLists.txt
│   ├── Implementation.cc
│   └── include
│       ├── Header1.h
│       └── Header2.h
└── test
    ├── CMakeLists.txt
    └── Test.cc
```

## Cmake lists

### `CmakeLists.txt` in the root path

```cmake
# CMake configuration file of WebServer

cmake_minimum_required(VERSION 3.10)

# set language std
set(CMAKE_CXX_STANDARD 20)
# set language std and disable fallback to a previous version
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# Disable CXX Syntax EXTENSIONS
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

project(WebServer
        VERSION 0.1
        DESCRIPTION "WebServer"
        LANGUAGES CXX
        )

# People keep running CMake in the wrong folder, completely nuking their project or creating weird bugs.
# This checks if you're running CMake from a folder that already has CMakeLists.txt.
# Importantly, this catches the common case of running it from the root directory.
file(TO_CMAKE_PATH "${PROJECT_BINARY_DIR}/CMakeLists.txt" PATH_TO_CMAKELISTS_TXT)
if (EXISTS "${PATH_TO_CMAKELISTS_TXT}")
    message(FATAL_ERROR "Run CMake from a build subdirectory! \"mkdir build ; cd build ; cmake .. \" \
    Some junk files were created in this folder (CMakeCache.txt, CMakeFiles); you should delete those.")
endif ()

######################################################################################################################
# COMPILER SETUP
######################################################################################################################

include_directories("src/include")

set(BASIC_CXXFLAGS "-O2 -g -pipe -fPIC -Wall -Wextra -Werror -pedantic-errors -pthread")
set(EXTRA_WARNINGS "-Wnon-virtual-dtor -Wold-style-cast -Woverloaded-virtual -Wsign-promo -Wswitch-default -Wfloat-equal -Wshadow -Wcast-qual -Wextra-semi -Wno-unused-parameter -Wno-attributes")

# Compiler flags.
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${BASIC_CXXFLAGS} ${EXTRA_WARNINGS}")
# cmake -DCMAKE_BUILD_TYPE=DEBUG ..
# set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0 -g -fno-omit-frame-pointer -fno-optimize-sibling-calls")
set(CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} -fPIC")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -fPIC")
set(CMAKE_STATIC_LINKER_FLAGS "${CMAKE_STATIC_LINKER_FLAGS} -fPIC")

set(GCC_COVERAGE_LINK_FLAGS "-fPIC")
message(STATUS "CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
message(STATUS "CMAKE_CXX_FLAGS_DEBUG: ${CMAKE_CXX_FLAGS_DEBUG}")
message(STATUS "CMAKE_EXE_LINKER_FLAGS: ${CMAKE_EXE_LINKER_FLAGS}")
message(STATUS "CMAKE_SHARED_LINKER_FLAGS: ${CMAKE_SHARED_LINKER_FLAGS}")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

add_subdirectory(src)
add_subdirectory(test)

######################################################################################################################
# COMPILER END
######################################################################################################################
```

### `CmakeLists.txt` in folder src

```cmake
file(GLOB resources *.cc)
set(server_shared_name server_shared)
add_library(${server_shared_name} SHARED ${resources})
```

### `CmakeLists.txt` in folder test

```cmake
file(GLOB test_resources *.cc)

foreach(test_source ${test_resources})
    # Create binary names from filenames
    message(STATUS "test file path ${test_source}")

    get_filename_component(test_filename ${test_source} NAME)
    message(STATUS "test filename ${test_filename}")

    string(REPLACE ".cc" "" test_name ${test_filename})
    message(STATUS "test name ${test_name}")
    
    add_executable(${test_name} ${test_source})
    target_link_libraries(${test_name} server_shared pthread)
endforeach(test_source ${test_resources})
```
