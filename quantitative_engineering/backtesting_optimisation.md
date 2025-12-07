---
sidebar_position: 3
title: "Backtesting Optmisation"
description: "Robust Backtesting and Optmisation, Building a Graph-Based Pipeline for Forex Scalping Strategies"
tags: ["Quantitative Analysis", "Backtesting"]
draft: false
---

Over the last few years, I've been exploring how to backtest with large CSV files containing raw tick data on AWS, at scale, using a range of fixed parameters.

I've come to realise it's not efficient to run through years of CSV tick data with fix parameters. It takes a long time and isn't efficient. 

I've transitioned to using a QuestDB database and introducing filters with multiple walk-forward stages in backtesting, to ensure only robust strategies advance through testing periods before final validation at full tick resolution. 

I'm going to explain how I've set up my backtesting environment to be as optimal and efficient as possible, sharing results and insight.

In the past, I have stored years of raw tick data on S3, and built pre-loaded AMIs, deploying thousands of instances to pull all of the data and evaluate the performance of one test case. It would scale up quickly on AWS, thousands of EC2's reporting to Elasticsearch and terminating. I'd then take these results into the live market, injecting the exact same strategy into the live environment. It was effective at the purpose of backtesting at scale but not efficient at identifying edge.

Through this time, I've tested hundreds of strategies across the last 14 years of raw tick data from FOREX, Indices, and Commodoties. I've taken them into the live market and make a lot of discoveries.

Firstly, previous results didn't align with real market conditions. My backtesting results had created artificially inflated performance that doesn't hold up in live trading. I was brute forcing through hundreds of thousands of parameter combinations to identify market gaps at a 0.2% success rate, which quickly degraded in the live environment.

I'm now redesigning my backtesting approach to use a graph-based neighbourhood optimisation method that tests parameter "neighbours" to find stable performance plateaus rather than fragile peaks, in addition to the filtering and walk-forward validation.

```
Initialise Grid → Test → UpdateResultAsync() → Classify
                                                  ↓
                            ┌─────────────────────┼─────────────────────┐
                            ↓                     ↓                     ↓
                          WINNER               NEUTRAL                LOSER
                            ↓                     ↓                     ↓
                        Trigger Stage 2         Record           Prune neighbors
                        (Optimisation)                           (Scorched Earth)
                            ↓                                          
                RunOptimisationLoopAsync() → Find new candidates 
```

<img src="/static/backtesting_optimisation/graphing_backtesting_runs.png" />

:::info Work in progress - December 2025

This is a live article. I write it all myself. There's a lot to describe and visualise properly.

I will post updates on X https://x.com/mccaffersuk and Threads https://www.threads.com/@mccaffers

:::