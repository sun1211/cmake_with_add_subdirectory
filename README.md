# How to Set up a CMakeLists.txt with add_subdirectories

## Structure of project
```
.
├── CMakeLists.txt
├── external //We simulate that code is provided by an "external" library outside of src
│   ├── CMakeLists.txt
│   ├── conversion.cpp
│   ├── conversion.hpp
│   └── README.md
├── src
│   ├── CMakeLists.txt
│   ├── evolution   //propagates the system in a time step
│   │   ├── CMakeLists.txt
│   │   ├── evolution.cpp
│   │   └── evolution.hpp
│   ├── initial    //produces the initial state
│   │   ├── CMakeLists.txt
│   │   ├── initial.cpp
│   │   └── initial.hpp
│   ├── io   //contains a function to print a row
│   │   ├── CMakeLists.txt
│   │   ├── io.cpp
│   │   └── io.hpp
│   ├── main.cpp      //the main function
│   └── parser   //parses the command-line input
│       ├── CMakeLists.txt
│       ├── parser.cpp
│       └── parser.hpp
└── tests  //contains two unit tests using the Catch2 library
    ├── catch.hpp
    ├── CMakeLists.txt
    └── test.cpp
```
## How to do it

**1. The top-level CMakeLists.txt is very similar to Recipe 1, Code reuse with functions and macros**
```
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
  
project(recipe-07 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(GNUInstallDirs)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY
  ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY
  ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY
  ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})

# defines targets and sources
add_subdirectory(src)

# contains an "external" library we will link to
add_subdirectory(external)

# enable testing and define tests
enable_testing()
add_subdirectory(tests)
```

**2.Targets and sources are defined in src/CMakeLists.txt (except the conversion target)**
```
add_executable(automata main.cpp)
  
add_subdirectory(evolution)
add_subdirectory(initial)
add_subdirectory(io)
add_subdirectory(parser)

target_link_libraries(automata
  PRIVATE
    conversion
    evolution
    initial
    io
    parser
  )
  ```
**3.The conversion library is defined in external/CMakeLists.txt**
```
add_library(conversion "")

target_sources(conversion
  PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}/conversion.cpp
  PUBLIC
    ${CMAKE_CURRENT_LIST_DIR}/conversion.hpp
  )

target_include_directories(conversion
  PUBLIC
    ${CMAKE_CURRENT_LIST_DIR}
  )
  ```
**4.The src/CMakeLists.txt file adds further subdirectories, which in turn contain CMakeLists.txt files. They are all similar in structure; src/evolution/CMakeLists.txt contains the following:**
```
add_library(evolution "")

target_sources(evolution
  PRIVATE
    evolution.cpp
  PUBLIC
    ${CMAKE_CURRENT_LIST_DIR}/evolution.hpp
  )
target_include_directories(evolution
  PUBLIC
    ${CMAKE_CURRENT_LIST_DIR}
  )
  ```
**5.The unit tests are registered in tests/CMakeLists.txt**
```
add_executable(cpp_test test.cpp)

target_link_libraries(cpp_test evolution)

add_test(
  NAME
    test_evolution
  COMMAND
    $<TARGET_FILE:cpp_test>
  )
  ```
  ## How to run it
```
$ mkdir -p build
$ cd build
$ cmake ..
$ cmake --build .
```