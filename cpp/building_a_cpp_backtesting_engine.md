---
sidebar_position: 3
title: "C++ Backtesting System"
description: "Rebuilding my backtesting system in C++ for performance at scale"
tags: ["C++", "Quantitative Engineering"]
---

## Introduction

Following on from my previous post, [Building a Backtesting Engine in C#](/quantitative_engineering/building_a_backtesting_system/), I decided to take the next step and rebuild the entire system in C++ to improve performance. 

:::info Work in progress - August 2026

This is a live document, I’m making frequent changes that may make some of this outdated. I will frequently update.

I will post updates on X https://x.com/ryanmccaffers

:::

### Why C++ 
C++ offers superior performance and lower-level control over system resources, which can lead to significant speed improvements and reduced memory usage. C++ is a superset of C which can be compiled natively for various platforms.

### Source code
I'm going to explain my work as I go, and for those interested in exploring the code in full, you can find the codebase on [GitHub](https://github.com/mccaffers/backtesting-engine-cpp)

## One binary, six subcommands

What started as a single backtesting loop has grown into a full trading pipeline. The engine is now C++23 (modules, `import std;`) and compiles to one binary that dispatches on its first argument:

* `ingest` receives a UDP tick stream and writes it to QuestDB
* `load` expands a strategy parameter sweep and queues it in Redis
* `run` drains the queue, backtests against QuestDB tick data, and reports results to Elasticsearch
* `live` takes the winning backtests from Elasticsearch and trades them live via the IG REST API
* `tracking` receives the IG account's deal updates over UDP and logs each one
* `positions` mirrors the IG account's open positions into Redis every minute

`main.cpp` is just a router now, each subcommand lives in its own module:

```cpp
import std;

import loadCommand;
import runCommand;
import ingestCommand;
import liveCommand;
import trackingCommand;
import positionsCommand;

int main(const int argc, const char* argv[]) {

    if (argc < 2) {
        std::println(std::cerr, "Error: missing subcommand");
        return 1;
    }

    const std::string_view subcommand = argv[1];

    if (subcommand == "load") return LoadCommand::run(argc, argv);
    if (subcommand == "run") return RunCommand::run(argc, argv);
    if (subcommand == "ingest") return IngestCommand::run(argc, argv);
    if (subcommand == "live") return LiveCommand::run(argc, argv);
    if (subcommand == "tracking") return TrackingCommand::run(argc, argv);
    if (subcommand == "positions") return PositionsCommand::run(argc, argv);

    std::println(std::cerr, "Error: unknown subcommand '{}'.", subcommand);
    return 1;
}
```

## Building C++

To begin with I needed a build automation tool to build the C++ application. I want to build the application on demand per platform as I'll be using multiple different operating systems (Linux on AWS and developing on a MacBook Pro). There are several options for build systems,

* Manual, roll your own bash scripts (with compiler commands directly)
* Make
* CMake

I've decided to use CMake because I want to build a large backtesting system and CMake can incorporate several dozen classes and interfaces in one automation script. It has cross platform compatibility and significantly simplifies the process.

Since the first version of this post, the project has moved from C++20 headers to C++23 modules with `import std;`, and that changes the toolchain requirements. Modules need the Ninja generator, and `import std;` needs a Clang whose libc++ ships a `std` module. Apple Clang doesn't, so on macOS I use Homebrew LLVM, and on Linux I use Clang with libc++ from apt.llvm.org. GCC is out for now.

Snippet of my `CMakeLists.txt`, which globs the source and module files, builds them into a static library, and links the executable against it:

```cmake
cmake_minimum_required(VERSION 3.30)
project(BacktestingEngine)

# Set the C++ standard
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Collect all .cpp files in the source directory
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "source/*.cpp")

# main.cpp belongs to the executable only
list(REMOVE_ITEM SOURCES ${CMAKE_SOURCE_DIR}/source/main.cpp)

# Create a library of the project's code
add_library(BacktestingEngineLib STATIC ${SOURCES})

# C++23 module interface units (.cppm files)
file(GLOB_RECURSE MODULE_INTERFACES CONFIGURE_DEPENDS
    "${CMAKE_SOURCE_DIR}/source/*.cppm")
target_sources(BacktestingEngineLib
    PUBLIC FILE_SET CXX_MODULES FILES
    ${MODULE_INTERFACES})

# Enable `import std;` for this target
set_target_properties(BacktestingEngineLib PROPERTIES CXX_MODULE_STD ON)
target_compile_features(BacktestingEngineLib PUBLIC cxx_std_23)

# Main executable
add_executable(BacktestingEngine source/main.cpp)
target_link_libraries(BacktestingEngine BacktestingEngineLib)
```

The real file is longer than this, `import std;` is still behind CMake's experimental flag, so there is a block pinning the activation UUID per CMake version, plus workarounds to help CMake find libc++'s module manifest on each platform. It also links the external dependencies: Boost (Redis and shared memory IPC), libpqxx for QuestDB's Postgres wire protocol, OpenSSL, libcurl, OpenMP, and the AWS SDK for DynamoDB.

### C++23 modules

In the first version of this project I used header files, and noted at the time that they share some similarities with interfaces in managed languages. I've since converted the codebase to C++23 modules. Each `.cppm` file is a module interface unit, it declares what it exports and imports what it needs, so there's no more textual `#include` of my own code and no include guards. A few headers remain where third party libraries (Boost.Asio, libcurl) are wrapped behind an implementation file to keep their macros out of the module builds.

### Project structure

I've structured the project with a directory per subcommand, shared code, tests and scripts:

```bash
.
├── CMakeLists.txt
├── external      # Vendored dependencies (Catch2, libpqxx, boost-decimal, AWS SDK)
├── scripts       # Build, test and per-subcommand wrapper scripts
├── source
│   ├── main.cpp  # Subcommand router
│   ├── ingest    # UDP tick receiver, QuestDB ILP writer
│   ├── load      # Sweep expansion and Redis queueing
│   ├── run       # Backtest worker
│   ├── live      # Live trading via the IG REST API
│   ├── tracking  # UDP deal receiver
│   ├── positions # IG position book mirror
│   ├── strategies # IStrategy, the factory and the strategies
│   └── shared    # Models, utilities, Redis/QuestDB/IG/AWS clients
└── tests         # Catch2 unit tests
```

### Execution

I'm still working with bash scripts on top of CMake. `scripts/build_dep.sh` builds the vendored dependencies once (libpqxx and the AWS SDK, which is enormous and deliberately kept out of the main build), and `scripts/build.sh` selects the right toolchain per platform and runs the configure and compile:

```bash
cmake .. -G Ninja -Wno-dev \
  "${TOOLCHAIN_ARGS[@]}" \
  -DCMAKE_CXX_STANDARD=23 \
  -DCMAKE_BUILD_TYPE=Release

cmake --build . --parallel "$JOBS"
```

There is then a wrapper script per subcommand (`load.sh`, `run.sh`, `ingest.sh`, `live.sh`) that validates the required environment variables and checks the services are reachable before launching. Getting started looks like this:

```bash
git clone --recurse-submodules https://github.com/mccaffers/backtesting-engine-cpp
cd backtesting-engine-cpp

bash ./scripts/build.sh    # CMake + Ninja + Clang/libc++
bash ./scripts/test.sh     # Catch2 tests via ctest

# with Redis, QuestDB and Elasticsearch running:
./build/BacktestingEngine load random    # queue a sweep
./build/BacktestingEngine run localhost  # drain and backtest
```

## Ingest of tick data

In the first version of this system I parsed yearly CSV files with `std::getline`, streaming the lines into a vector of price records on a background thread. That worked, but the tick store has since moved into QuestDB, a time series database, which lets multiple machines share one dataset and lets the database do the date windowing for me. I covered optimising the read path in [For Loop Optimisation: Parsing Tick Data at Speed](/cpp/for_loop_optimisation/).

The write path is the `ingest` subcommand. An external streamer publishes ticks as fixed 40-byte little-endian UDP packets (`bid` f64, `ask` f64, timestamp in microseconds i64, symbol `char[16]`). `ingest` binds a UDP socket, decodes and validates each packet, and batch-writes to QuestDB using ILP over HTTP:

```cpp
net::UdpReceiver receiver(
    bindAddr, bindPort, [&](std::span<const std::byte> bytes) {
        const auto tick = tick_packet::decodeTick(bytes);
        if (!tick) {
            dropped.fetch_add(1, std::memory_order_relaxed);
            return;
        }
        // ILP line: table = symbol, fields ask/bid, designated timestamp
        const auto tsNanos = std::chrono::duration_cast<std::chrono::nanoseconds>(
                                 tick->timestamp.time_since_epoch())
                                 .count();
        writer.enqueueLine(std::format("{} ask={}i,bid={}i {}\n",
                                       tick->symbol, tick->ask, tick->bid, tsNanos));
        received.fetch_add(1, std::memory_order_relaxed);
    });
```

Writes are buffered through a bounded queue, batches of 1,000 lines or 100ms, dropping the oldest lines at a million so QuestDB being down can't take out the receiver. Malformed, non-finite or unknown-symbol ticks are dropped at decode. The columns written (`ask`, `bid`, `timestamp`, one table per symbol) are exactly what the `run` subcommand reads back.

## Queueing backtests with Redis

With thousands of parameter combinations to test, I need a work queue. The `load` subcommand expands a strategy parameter sweep (combinations by symbol groups) into individual backtest payloads and pushes them into Redis as base64 encoded JSON. Before touching Redis it prints the total sweep size and waits for confirmation on stdin, so I can't accidentally queue an enormous grid.

| Redis key | Type | Contents |
| --- | --- | --- |
| `BACKTESTING_QUEUE_RUN` | list | Run descriptors (symbols, date window, risk limits) |
| `BACKTESTING_QUEUE_STRATEGY:<RUN_ID>` | list | Payload key names for that run |
| `BACKTESTING_QUEUE_STRATEGY_PAYLOAD:<RUN_ID>:<UUID>` | string | One strategy config, with a 7 day safety net TTL |

The write order matters: payloads exist before their names are visible, and the strategy list is complete before the run is advertised, so a worker that discovers a run can immediately drain it.

## Running backtests

The `run` subcommand drains the queue and executes the backtests. Run descriptors are peeked, never popped, so any number of `run` processes across machines can discover the same run and drain its strategy list in parallel. `RPOP` hands each strategy to exactly one worker and `GETDEL` consumes each payload exactly once. A worker dying mid-run leaves the descriptor in place for the next peek, so there's no distributed lock to manage. Backtests are submitted to a thread pool sized at 80% of hardware threads.

Tick data is cached across runs. Rather than paying a full QuestDB load per backtest, the worker keeps one months-deep superset of ticks per symbol set, and each date window is served as a contiguous slice of that superset, found by binary search on month boundaries that QuestDB computes itself.

Each strategy walks forward through history on a rolling window ladder: three months at offset zero, then offset three, then offset six, then the full nine month history. A run that completes re-queues itself on the next window automatically. Results go to Elasticsearch in weekly indices, and completed full-history runs land in a winners index that the live engine selects from.

## Creating a Strategy Class

Every tick event still drives the strategy, but the interface has evolved from a single `shouldTrade` function into two hooks. `decide` returns an optional entry direction, and `during` runs on every tick for bar building and trade management:

```cpp
export module strategy;

import std;
import barStore;      // bars::BarStore, the shared per-symbol bar histories
import priceData;     // PriceData
import trade;         // Direction
import tradeManager;  // TradeManager

export class IStrategy {
public:
    virtual ~IStrategy() = default;

    // Entry signal. Returns std::nullopt to mean "no trade".
    virtual std::optional<Direction> decide(const PriceData& tick,
                                            const bars::BarStore& barStore) = 0;

    // Called every tick, for bar building and trade management
    virtual void during(const PriceData& price,
                        const bars::BarStore& barStore,
                        TradeManager& tradeManager) = 0;
};
```

The two-hook split exists because the run loop gates `decide` behind "no open trade for this symbol", so a strategy managing an open position only ever observes the market from `during`. The `virtual ~IStrategy() = default;` destructor is still mandatory, deleting a derived strategy through a base pointer without it would skip the derived destructor and leak resources.

A single factory (`strategies::makeStrategy`) constructs strategies for both backtesting and live trading, so a strategy config travels byte-identical (same UUID) from sweep, to queue, to backtest, to live selection. There are nine strategies at the moment:

* `randomStrategy`, random entries over a stop loss and take profit grid, a baseline harness rather than a candidate edge
* `ohlcBreakout`, multi-bar range breakout with an EMA trend filter and a time cap exit
* `fvg`, fair value gap retracement with a higher timeframe trend filter
* `keltnerFade`, mean reversion, fading stretches beyond a volatility band
* `sessionRangeBreakout`, London open breakout of the Asian session range
* `nyOpenRangeBreakout`, the New York sibling, breakout of the pre-open range after the 09:30 equities open
* `squeezeBreakout`, volatility contraction, breaks of a single compressed bar
* `liquiditySweepReversal`, fades the failed breakout, a wick through an old swing high or low that closes back on the original side
* `rangeVelocity`, momentum on range bars, entering when consecutive bars all point one way and each formed faster than the recent norm

Adding a strategy is one branch in the factory, a sweep and config-factory pair under `source/load/config/`, and a branch in the `load` command.

## Unit tests

I originally pulled GoogleTest into the project with CMake's `FetchContent`, which needs an internet connection at configure time. I've since moved to Catch2, vendored as a git submodule in `external/`, so there's no network dependency and the same version builds everywhere (useful when I'm working while mobile with my MacBook).

> Catch2 Repository, https://github.com/catchorg/Catch2

The tests live in `/tests`, one translation unit per area under test, and each one links the project library and imports its modules directly:

```cmake
add_executable(unit_tests
    ema.cpp
    atr.cpp
    marketHours.cpp
    tradeManager.cpp
    # ... one file per area under test
)

target_link_libraries(unit_tests PRIVATE
    BacktestingEngineLib
    Catch2::Catch2WithMain)

# Match the library's module configuration
set_target_properties(unit_tests PROPERTIES CXX_MODULE_STD ON)
target_compile_features(unit_tests PRIVATE cxx_std_23)

# Register every TEST_CASE with CTest
include(${Catch2_SOURCE_DIR}/extras/Catch.cmake)
catch_discover_tests(unit_tests)
```

Catch2's `TEST_CASE` and `SECTION` macros make the tests read nicely, each section runs independently from the top of the test case:

```cpp
#include <catch2/catch_test_macros.hpp>

#include <cstdint>
#include <vector>

import ema;

TEST_CASE("ema::calculate matches the C# reference shape", "[ema]") {
    SECTION("pads with zeros, seeds with the SMA, then follows the recurrence") {
        const std::vector<std::int32_t> prices{10, 20, 30, 40, 50};
        const auto out = ema::calculate(prices, 3);

        REQUIRE(out.size() == prices.size());
        CHECK(out[0] == 0);
        CHECK(out[1] == 0);
        CHECK(out[2] == 20);  // SMA(10, 20, 30)
        CHECK(out[3] == 30);  // 0.5*40 + 0.5*20
        CHECK(out[4] == 40);  // 0.5*50 + 0.5*30
    }
}
```

`scripts/test.sh` builds and runs the suite through ctest:

```bash
bash ./scripts/test.sh          # build + ctest --output-on-failure
CLEAN=1 bash ./scripts/test.sh  # force a clean reconfigure first
```

### Video demonstration of tests running



<video loop="true" autoplay="autoplay" muted autoplay playsinline class="video-auto rounded" width="100%">
<source src="/static/backtesting_cpp/terminal-tests.mp4" type="video/mp4" />
Your browser does not support video playback.
</video>

## Going live

The winners don't stay theoretical. The `live` subcommand pulls the winning full-history runs from Elasticsearch (above a minimum score, within a maximum drawdown), instantiates their strategies through the same factory, and routes a live UDP tick stream to them, one worker thread per symbol. Orders are placed with the IG REST API, with session credentials pulled from DynamoDB.

Every entry is gated: Redis trade locks, position caps, and portfolio cluster exposure limits, all of which fail closed if Redis is unreachable. Closes are the opposite and fail open, an unsent close leaves live exposure, which is the one outcome worse than an extra request. The broker owns positions and exits, the engine mirrors the position book from Redis and only emits opens and closes.

Two supporting subcommands keep the live picture honest: `tracking` logs every deal update the account produces (256-byte UDP deal packets from IG's Lightstreamer feed), and `positions` mirrors the broker's own open position book into Redis every minute, keeping the broker the source of truth for what is actually open.

I'll expand on the live trading architecture in a future post.

Work in progress - More to follow!
