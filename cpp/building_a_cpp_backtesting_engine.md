---
sidebar_position: 3
title: "C++ Backtesting System"
description: "Rebuilding my backtesting system in C++ for performance at scale"
tags: ["Quantitative Engineering", ".NET trading", "visualising trading", "canvasjs", "ohlc"]
---

## Introduction

Following on from my previous post, [Building a Backtesting Engine in C#](/quantitative_engineering/building_a_backtesting_system/), I decided to take the next step and rebuild the entire system in C++ to improve performance. 

:::info Work in progress - November 2025

This is a live document, I’m making frequent changes that may make some of this outdated. I will frequently update.

I will post updates on X https://x.com/mccaffersuk and Threads https://www.threads.com/@mccaffers

:::

### Why C++ 
C++ offers superior performance and lower-level control over system resources, which can lead to significant speed improvements and reduced memory usage. C++ is a superset of C which can be compiled natively for various platforms.

### Source code
I'm going to explain my work as I go, and for those interested in exploring the code in full, you can find the codebase on [GitHub](https://github.com/mccaffers/backtesting-engine-cpp)

## Building C++

To begin with I need a build automation tool to build the C++ application. I want to build the application on demand per platform as I'll be using multiple different operating systems (Linux on AWS, Windows Servers and developing on a MacBook Pro). There are several options for build systems,

* Manual, roll your own bash scripts (with gcc commands `gcc -std=c++20 program.cpp`)
* Make
* CMake

I've decided to use CMake because I want to build a large backtesting system and CMake can incorporate several dozen classes and interfaces in one automation script. It has cross platform compatability and significantly simplifies the process.

To do this you need gcc installed (it's installed by default on Linux [*most popular flavours*] and macOS [*actually Clang/LLVM which provides a gcc command*]).

Snippet of my `CMakeLists.txt` which defines the headers to include and source/test files to compile 
```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 20)

# Include directories
include_directories(
    ${CMAKE_SOURCE_DIR}/include
    ${CMAKE_SOURCE_DIR}/include/Strategies
    ${CMAKE_SOURCE_DIR}/include/FileManagement
)

# Source files
set(SOURCES
    src/PriceRecord.cpp
    src/Account.cpp
    src/CSVParser.cpp
    src/Strategies/SimpleMovingAverageStrategy.cpp
    src/Strategies/RandomStrategy.cpp
    src/FileManagement/CSVStream.cpp
)

# Create a library of your project's code
add_library(MyProjectLib STATIC ${SOURCES})

# Main executable
add_executable(MyProgram src/program.cpp)
target_link_libraries(MyProgram MyProjectLib)
```

I've going to structure my project with a resources, headers, a sources and test directory:

```bash
.
├── CMakeLists.txt
├── include # All Headers
│   ├── *
├── readme.md
├── resources # Contains tick data
├── run.sh
├── src
│   ├── * # All source files
└── tests
```

I'll refine this as I go and may move the header files next to the source files, depending on what makes most sense as it grows. (Clear separation of interface and implementation vs Keeps related files together, which can be more intuitive). 

C++ header files share some similarities with interfaces in managed languages, but they are not exactly the same. I'll explain more as I continue.

### Execution

I'll use a bash script to execute the program with CMake. I'm working with a bash script on top of CMake as I want to clean the build folder after local use, and to define multiple build and test scripts.

Snippet of my inital execute script (`./scripts/run.sh`):

```bash
#!/bin/bash

# Variables
BUILD_DIR="build"
EXECUTABLE_NAME="MyProgram"
export DATA_FILE_PATH="resources/EURUSD/2020.csv"

# Step 1: Create a build directory if it doesn't exist
if [ ! -d "$BUILD_DIR" ]; then
    mkdir "$BUILD_DIR"
fi

# Step 2: Navigate to the build directory
cd "$BUILD_DIR" || exit

# Step 3: Run CMake to configure the project
cmake ..

# Step 4: Compile the project
cmake --build .

# Step 5: Navigate back to the root directory
cd ..

# Debug: Check if the executable exists
if [ -f "$BUILD_DIR/$EXECUTABLE_NAME" ]; then
    echo "Executable $EXECUTABLE_NAME found in $BUILD_DIR."
else
    echo "Error: Executable $EXECUTABLE_NAME not found in $BUILD_DIR."
    ls -la "$BUILD_DIR" # List the contents of the build directory for debugging
    exit 1
fi

# Step 6: Run the executable from the root directory
./"$BUILD_DIR/$EXECUTABLE_NAME"

# # Step 7: Clean up by removing the build directory
rm -rf "$BUILD_DIR"

# # Optional: Print a success message
echo "Build, run, and cleanup complete!"
```

## Ingest of tick data

To begin with, I'm going to focus on the ingest of tick data from a CSV file using `std::getline(file, line)`.

> The C++ Standard Library, often referred to as the C++ STL (Standard Template Library), is a collection of powerful, reusable components that form an integral part of the C++ language. You can shorten this to `getline(file,line)` if import the std namespace (`using namespace std;`)

I'm working with CSV files that are chunked into yearly datasets and I want to work with multiple years of data. To work with these files I'm going to use `threads` and `buffers` to concurrently ingest and process the data.

I'm using `std::ifstream` for file input and `std::stringstream` to stream the lines of data, string manipulation, converting data into a vector of `PriceRecord` objects.

> `std::ifstream` is used for reading from files. It's part of the `<fstream>` header

> `std::stringstream` is a stream class that uses a string buffer. It's part of the `<sstream>` header

```cpp
std::thread([this]() {
    std::string line;
    std::getline(file, line); // Skip header

    while (std::getline(file, line)) {
        std::stringstream ss(line);
        std::string utc, askPriceStr, bidPriceStr;

        std::getline(ss, utc, ',');
        std::getline(ss, askPriceStr, ',');
        std::getline(ss, bidPriceStr, ',');

        double askPrice = std::stod(askPriceStr);
        double bidPrice = std::stod(bidPriceStr);

        PriceRecord record(utc, askPrice, bidPrice);

        {
            std::lock_guard<std::mutex> lock(mtx);
            buffer.push(record);
        }
        cv.notify_one();
    }

    {
        std::lock_guard<std::mutex> lock(mtx);
        done = true;
    }
    cv.notify_all();
}).detach();

```

## Creating a Strategy Class

Every tick event will trigger the Backtesting Engine to call the `shouldTrade` function of the strategy. The strategy will then decide whether to buy or sell.

```cpp
#ifndef STRATEGY_H
#define STRATEGY_H

#include "PriceRecord.h"
#include <vector>

class Strategy {
public:

    // Virtual destructor for the Strategy class
    // Destructors are special member functions that are called when an object is destroyed or goes out of scope.
    // They are responsible for cleaning up any resources that the object may have acquired during its lifetime
    // When you have a base class (like Strategy) and derived classes (like various strategies), and you're using polymorphism (i.e., handling derived class objects through base class pointers), you need a virtual destructor in the base class to clean up objects that may have been accquired during it's lifetime
    virtual ~Strategy() = default;


    // Trading decision
    virtual bool shouldTrade(const std::vector<PriceRecord>& priceHistory) = 0;
};

#endif // STRATEGY_H
```

## Unit tests

I've added GoogleTest is Google's C++ testing and mocking framework to the project to test the Account, PriceRecord, and RandomStrategy class.

There are lots of testing frameworks to pick from, I looked at Gtest, Catch2 and CppUnit

> Google Test Repository, https://github.com/google/googletest

> Catch2 Repository, https://github.com/catchorg/Catch2

> CppUnit Repository, https://github.com/Ultimaker/CppUnit

I'm using `FetchContent_Declare` in the CMakeList.txt to dynamically pull Google C++ Test Framework to build unit tests:

```cmake
set(FETCHCONTENT_UPDATES_DISCONNECTED ON)

# GoogleTest is Google's C++ testing and mocking framework
include(FetchContent)
FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG release-1.11.0
)
FetchContent_MakeAvailable(googletest)

# Enable testing for current directory and below
enable_testing()

# Test sources
set(TEST_SOURCES
    tests/PriceRecordTest.cpp
)

# Test executable
add_executable(MyProgramTests ${TEST_SOURCES})
target_link_libraries(MyProgramTests MyProjectLib gtest gtest_main)

# Add the test
add_test(NAME MyProgramTests COMMAND MyProgramTests)
```

To note, to use `FetchContent_Declare`, the script requires an internet connection. As I'm working on a local machine, I've added the `set(FETCHCONTENT_UPDATES_DISCONNECTED ON)` to the CMakeLists.txt to cache the dependency so I can work while mobile with my MacBook.

Then added tests into /tests, 

```cpp
#include <gtest/gtest.h>
#include "PriceRecord.h"

TEST(PriceRecordTest, ConstructorAndGetters) {
    PriceRecord record("2023-09-02 10:00:00", 100.5, 100.0);
    
    EXPECT_EQ(record.getUTC(), "2023-09-02 10:00:00");
    EXPECT_DOUBLE_EQ(record.getAskPrice(), 100.5);
    EXPECT_DOUBLE_EQ(record.getBidPrice(), 100.0);
}
```

I've created a test bash script `./scripts/tests.sh` to build and run the tests.

### Video demonstration of tests running



<video loop="true" autoplay="autoplay" muted autoplay playsinline class="video-auto rounded" width="100%">
<source src="/static/backtesting_cpp/terminal-tests.mp4" type="video/mp4" />
Your browser does not support video playback.
</video>



Work in progress - More to follow!
