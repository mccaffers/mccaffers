---
sidebar_position: 3
title: "Backtesting Optmisation"
description: "Robust Backtesting and Optmisation, Building a Graph-Based Pipeline for Forex Scalping Strategies"
tags: ["Quantitative Analysis", "Backtesting"]
draft: false
---

Over the last few years, I've been exploring how to backtest at scale with large CSV files on AWS at scale, using fixed take-profits and stop-losses.

I've come to realise it's not efficient to run through years of CSV tick data with fix parameters. I've transitioned to using a QuestDB database with raw tick data, introducing filters with multiple walk-forward stages to ensure only robust strategies advance through testing periods before final validation at full tick resolution. 

I'm going to explain how I've set up my backtesting environment to be as optimal and efficient as possible, sharing results and insight.

I previously used to place years of raw tick data on S3, use pre-loaded AMIs, and launch instances that would pull various years of data (let's say the last 10 years of tick data) and evaluate a set of parameters. I would scale up to have thousands of EC2s running, reporting to ElasticSearch. I'd then take these results into the live market, injecting the exact same strategy into the live environment.

Through this time, I've tested hundreds of strategies in live, across FOREX, indices, and commodities, and discovered a lot! 

My previous results didn't align with real market conditions. My backtesting results had created artificially inflated performance that doesn't hold up in live trading. I was brute forcing through hundreds of thousands of parameter combinations to identify market gaps at a 0.2% success rate, which quickly degraded in the live environment.

I'm now redesigning my backtesting approach to use a graph-based neighbourhood optimisation method that tests parameter "neighbours" to find stable performance plateaus rather than fragile peaks, in addition to the filtering and walk-forward validation.

:::info Work in progress - November 2025

This is a live article. I write it all myself. There's a lot to describe and visualise properly.

I will post updates on X https://x.com/mccaffersuk and Threads https://www.threads.com/@mccaffers

:::