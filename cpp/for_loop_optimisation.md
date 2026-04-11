---
sidebar_position: 4
title: "For Loop Optimisation: Parsing Tick Data at Speed"
description: "Two approaches to parsing tick data in C++"
tags: ["C++", "Quantitative Engineering"]
---

## Two Loops, Two Philosophies

Following on from my previous post, [Building a Backtesting Engine in C++](/quantitative_engineering/building_a_cpp_backtesting_engine/), I started to evaluate data ingest performance from QuestDB. I want the C++ application to evaluate a strategy off a (git) branch, ingest the last *n* months of data, to spawn multiple sweeps as fast as possible.

I started off with a convenient loop, building a vector of `PriceData`.

## The Convenient Loop

```cpp
for (const auto& row : result) {
    double value1 = row[0].as<double>();
    double value2 = row[1].as<double>();
    std::string timestamp_str = row[2].as<std::string>();
    auto timestamp = Utilities::parseTimestamp(timestamp_str);
    results.emplace_back(value1, value2, timestamp);
}
```

This reads naturally. libpqxx's `as<double>()` and `as<std::string>()` handle the conversion for you. The problem is what's hidden behind that convenience:

- `as<std::string>()` constructs a heap-allocated string for every single timestamp, one malloc per row.
- `as<double>()` routes through libpqxx's general-purpose parsing machinery, which is locale-aware and may involve virtual dispatch.
- `parseTimestamp()` likely calls `timegm()` on every row to convert a calendar date into an epoch, an expensive syscall-adjacent operation.

## The Fast Loop

I then explored optimisations, using `string_view` to avoid heap allocations and parsing floats directly with `std::from_chars`.

```cpp
for (int i = 0; i < (int)result.size(); ++i) {
    const auto& row = result[i];
    double value1, value2;
    auto sv1 = row[0].view();
    auto sv2 = row[1].view();
    std::from_chars(sv1.data(), sv1.data() + sv1.size(), value1);
    std::from_chars(sv2.data(), sv2.data() + sv2.size(), value2);
    results[i] = PriceData(value1, value2, fastParseTimestamp(row[2].c_str()));
}
```

- `std::from_chars` parses floats directly from a `string_view`, no allocation, no locale, no virtual dispatch. It's one of the fastest float-parsing routines in the standard library.
- `fastParseTimestamp` uses `sscanf` to extract the date components and then caches the result of `timegm` per calendar day. For tick data where millions of rows share the same date, the expensive date-to-epoch conversion happens once, not millions of times. Subsequent rows simply add hours, minutes, and seconds to that cached base.
- Pre-sizing `results[i]` avoids repeated reallocation that `emplace_back` can cause if the vector isn't reserved.

## fastParseTimestamp

The timestamp caching is the most impactful optimisation in the fast loop, so it's worth looking at in full:

```cpp
static std::chrono::system_clock::time_point fastParseTimestamp(const char* ts) {
    int year, month, day, hour, min, sec, usec = 0;
    std::sscanf(ts, "%4d-%2d-%2d %2d:%2d:%2d.%d", &year, &month, &day, &hour, &min, &sec, &usec);

    // Cache timegm per date, tick data is time-ordered so date changes rarely
    static char cachedDate[11] = {};
    static time_t cachedEpoch = 0;
    if (std::memcmp(ts, cachedDate, 10) != 0) {
        std::memcpy(cachedDate, ts, 10);
        std::tm tm = {};
        tm.tm_year = year - 1900;
        tm.tm_mon  = month - 1;
        tm.tm_mday = day;
        tm.tm_isdst = 0;
        cachedEpoch = timegm(&tm);
    }

    time_t t = cachedEpoch + hour * 3600 + min * 60 + sec;
    return std::chrono::system_clock::from_time_t(t) + std::chrono::microseconds(usec);
}
```

The key insight is the `static` cache. `timegm()` converts a broken down calendar date into a Unix epoch. it's not a free operation, and calling it on every tick row is wasteful when tick data is time-ordered and millions of consecutive rows share the same date.

The cache works by comparing only the first 10 characters of the timestamp string (the `YYYY-MM-DD` portion) against the previously seen date using `memcmp`. If the date hasn't changed, `cachedEpoch` is reused directly and the hours, minutes, and seconds are simply added as an integer offset. `timegm()` is only called when the date actually rolls over typically once per trading day.

The function also handles sub-second precision via the `usec` field, returning a `std::chrono::system_clock::time_point` with microsecond resolution, which is the granularity needed for tick-level backtesting.

## Data Flow Comparison

<img src="/static/cpp/for_loop_flow.svg"/>

## When Does It Matter?

For general-purpose applications, the first loop is the right choice. It's readable, maintainable, and the performance cost is negligible.

But for high-frequency financial data market tick ingest, order book replay, backtesting pipelines. I'm routinely processing tens of millions of rows per session. At that scale, eliminating one heap allocation and one `timegm` call per row isn't premature optimisation. It's the difference between a pipeline that keeps up with the feed and one that doesn't.

The core insight is that the bottleneck is often not the obvious work (parsing a float), but the hidden overhead, allocating a string only to pass it to something that reads it and throws it away. Profiling tick data pipelines almost always surfaces string construction as a top contributor.
