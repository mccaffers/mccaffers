---
sidebar_position: 1
title: "Random Trading Strategies"
description: "Backtesting a Random Trading Strategy in various financial markets, Foreign Exchange, Indices, and Crypto, between 2004 and 2024"
tags: ["Quantitative Engineering", ".NET trading", "visualising trading", "canvasjs", "ohlc"]
---

In this post, I'm reviewing the performance of a Random Trading Strategy across multiple financial assets, testing hundreds of permutations, and exploring different exit strategies. The focus of this post revolves around a question that has consistently captured my curiosity...

<div class="frontpage-span" style={{ marginBottom: '40px' }}>Can you randomly trade the financial markets and profit?</div>

I'm using the [Backtesting Engine](/quantitative_engineering/building_a_backtesting_system/) to scale, document, and visualise performance using historical datasets from the two last decades, 2004 through to 2024. I've developed a Random Trading Strategy which will open a position in a random direction (a BUY or SELL trade) and is available to see on Github, 🔗 [C#  - Backtesting Engine / Random Strategy](https://github.com/mccaffers/backtesting-engine/blob/main/src/strategies/locked/Random/Random.cs)

## Test Scenarios

I'll test and experiment with a number of exit strategies:

### Fixed Targets

For the first test, I’ll set fixed targets for the Stop Loss (SL) and Take Profit (TP). Each trade will have a fixed point in which it will close the trade. This fixed point is based on a distance from the opening level.

### Range of Fixed Targets

The second experiment will scale up to execute multiple test runs. Each run will have a different fixed Stop Loss (SL) and Take Profit (TP), testing multiple permutations. As an example one run might have a Stop Loss and Take Profit of 20 points, while the next run might have a Stop Loss and Take Profit of 40. This allows you to compare the results across targets.

### Trailing Stop Loss

Exiting on a fixed initial STOP LOSS with a trailing STOP LOSS, without a TAKE PROFIT set. Each trade will have a fixed STOP LOSS, based on a distance from the opening level with a trailing SL. This will allow trades to run for as long as they can, as soon as they reverse the trailing SL will exit if the market moves against you.

### Higher Highers and Lower Lows

Exit when the price hits a recent Higher High or a Lower Low. Each trade will have a SL and TP linked to a recent HH or LL. Various timescales will be used to find a recent HH or LL.


## Market Asset Classes

I'm backtesting across several financial assets:

* Major Foreign Exchange Markets
  * EUR/USD, GBP/USD, AUD/USD, NZD/USD, USD/JPY, USD/CHF and USD/CAD

* Indices
  * STOXX Europe 50, UK FTSE100, USA30, S&P 500, NASDAQ, S&P/ASX 200, DAX, CAC 40, IBEX 35 and Nikkei 225

* Crypto Markets
  * BTCUSD and ETHUSD

* Bonds
  * UK Long Gilt, Euro Bund, US Treasury Bond
  
## Fixed Targets

* Random BUY or SELL position, at max frequency (as soon as the previous trade closes another trade opens)
* Every trade has a 30 pip Take Profit and Stop Loss from the opening position
* Trade with unlimited equity to review the strategy over the whole dataset

### Major Foreign Exchange Markets (2004 - 2024)

<img src="/static/randomly_trading/light/random-forex-from-2004.svg" />

All of the currencies go into negative equity quite early on, within 3 months and typically maintain a downwards trajectory.
Notable events:
1. There was an increase in failed trades following the Global Financial Crisis of 2007-2008. These trades were failing more often due to the increased volatilitly in the markets.
2. The `USDCHF` took a significant drop in account equity (to -120,000) in January 2015 when the Swiss National Bank removed it's currency peg of 1.20 to the euro, which caused an immediate 30% increase in value of the Swiss Franc. The voliatility caused significant losses as the system was trading a max frequency.

![](/static/randomly_trading/swissfranc.png)

### Indices (2014 - 2024)

<img src="/static/randomly_trading/light/random-indices-2014.svg"  />

All of the indices go into negative equity quite early on too. 

1. The JPN INDEX 225 (Nikkei 225) moves into negative equity and maintains a negative decline, which indicates that the index moves frequently and is quite volatile. 
2. USA30IDX, the Dow Jones Industrial Average (or simple the Dow) is a stock market index of 30 prominent companies in the United States, such as Apple, Boeing, Goldman Sachs, IBM, Intel, McDonalds, Microsoft, Nike, Visa, Walmart and more. There was huge volatility in the markets following the 2020 outbreak of COVID-19. The USAIDX30 felt the impact the most with this random trade strategy.

### Bonds (2020 - 2024)

<img src="/static/randomly_trading/light/random-bonds-fixed.svg"  />

## Range of Fixed Targets

Next, to scale up and test multiple Stop & Take Profit values with a realistic account (10,000).

* Every position will have a fixed STOP and TP, however there are multiple permutations, each deploying new experiments between 20 and 100 for both the SL and TP, so SL 20 TP 20, SL 20 TP 40, and so on.
* The system will trade at max frequency (as soon as the previous trade closes another trade opens)
* The system will trade with 10000 account equity
* Maximum drawdown 75% (trading run will be terminated if the account equity falls below 2500)

### Major Foreign Exchange Markets

* Major Foreign Exchange - EUR/USD, GBP/USD, AUD/USD, NZD/USD, USD/JPY, USD/CHF, USD/CAD
* Tick market data, 2004 to 2024
* 175 permutations

<img src="/static/randomly_trading/light/majorforex-variablestoplimit.svg"  />

### Indices

* 10 Indices - STOXX Europe 50, UK FTSE100, USA30, S&P 500, NASDAQ, S&P/ASX 200, DAX, CAC 40, IBEX 35, Nikkei 225
* Tick market data, 2014 to 2024
* 250 permutations

<img src="/static/randomly_trading/light/random-indices-variable.svg"  />

Theres a lot going on here, so lets break this down:

S&P 500

![](/static/randomly_trading/light/random-indices-sp500-variable.svg)

FTSE100

![](/static/randomly_trading/light/random-indices-ftse100-variable.svg)

STOXX 50

![](/static/randomly_trading/light/random-indices-stoxx-variable.svg)

S&P ASX 200 

![](/static/randomly_trading/light/random-indices-sp200asx-variable.svg)

CAC 40

![](/static/randomly_trading/light/random-indices-cac40-variable.svg)

NASDAQ

![](/static/randomly_trading/light/random-indices-nasdaq-variable.svg)

Nikkei 255

![](/static/randomly_trading/light/random-indices-nikkei255-variable.svg.svg)

IBEX 35

![](/static/randomly_trading/light/random-indices-ibex35-variable.svg)

### Bonds
* 3 Bonds - UK Long Gilt, Euro Bund, US Treasury Bond
* Tick market data, 2020 to 2024
* 75 permutations

<img  src="/static/randomly_trading/light/random-bonds-variable.svg"  />

### Cryptocurrencies Results
* 2 Cryptocurrencies - BTCUSD, ETHUSD
* Tick market data, 2018 to 2024
* 50 permutations

<img src="/static/randomly_trading/light/random-crypto-variable.svg"  />

## Trailing Stop Loss

A trailing stop loss is a stop loss that moves with the price. When the price moves in your favor it adjusts to keep reducing risk but also gives enough space for price fluctuations. If the price moves against you the stop loss will close out the trade. This can be applied in both ways, for a BUY/SELL. 

I’m also experimenting with a moving TP. This means there is no fixed value for a Take Profit, the position will continue as long as there is momentum in the trade direction.

<img class="trailingstoplosssvg" src="/static/randomly_trading/light/trailing-stoploss.drawio-diagram.svg"  />

### Major Foreign Exchange Markets

* Major Foreign Exchange - EUR/USD, GBP/USD, AUD/USD, NZD/USD, USD/JPY, USD/CHF, USD/CAD
* Tick market data, 2004 to 2024
* Trailing SL and moving TP, 100, 120, 140, 160, 180, 200
* 42 permutations

<img src="/static/randomly_trading/light/random-forex-trailingsl.svg"  />

### Indices Result

* 10 Indices - STOXX Europe 50, UK FTSE100, USA30, S&P 500, NASDAQ, S&P/ASX 200, DAX, CAC 40, IBEX 35, Nikkei 225
* Tick market data, 2014 to 2024
* Trailing SL and moving TP, 100, 120, 140, 160, 180, 200
* 60 permutations

<img src="/static/randomly_trading/light/random-indices-trailingsl.svg"  />

### Cryptocurrencies Results
* 2 Cryptocurrencies - BTCUSD, ETHUSD
* Tick market data, 2018 to 2024
* Trailing SL and moving TP, 100, 120, 140, 160, 180, 200
* 12 permutations

<img src="/static/randomly_trading/light/random-crypto-trailingsl.svg"  />

### Bonds Results
* 3 Bonds - US T-Bond, Europe Bund and UK Long Gilt
* Tick market data, 2018 to 2024
* Trailing SL and moving TP, 100, 120, 140, 160, 180, 200
* 18 permutations

<img src="/static/randomly_trading/light/random-bonds-traillingsl.svg"  />

The Bonds do not move as much to capture many trades using such a large moving SL (of more than 100). 

Lets test a smaller range:

* Trailing SL and moving TP, 20, 40, 60, 80, 100
* 15 permutations

<img src="/static/randomly_trading/light/random-bonds-trailingsl-smaller.svg"  />

## Recent Highs and Recent Lows

This exit strategy uses the recent highs and lows as Stop Loss and Take Profits. Various sample sizes of recent high and lows are captured and tested at different values.

* Continously trades, with a maximum of one open trade
* Random BUY or SELL order
* To exit a BUY trade
  * Recent high is set as the TAKE PROFIT, to the maximum of the range defined
  * Recent low is taken as the STOP LOSS, to the maximum of the last two OHLC objects (60 minutes)
* To exit a SELL trade
  * Recent low is set as the TAKE PROFIT, to the maximum of the range defined
  * Recent high is taken as the STOP LOSS, to the maximum of the last two OHLC objects (60 minutes)
* Multiple permutations are tested, with a range of 4 (2 hours), 6 (3 hours), 8 (4 hours) and 10 (5 hours)

<img src="/static/randomly_trading/light/random-recent-highlow-diagram.svg"  />

### Major Foreign Exchange Markets
* 28 permutations

<img src="/static/randomly_trading/light/random-forex-hhll.svg" />

### Indices
* 40 permutations

<img src="/static/randomly_trading/light/random-indices-hhll.svg" />

### Bonds
* 18 permutations

<img src="/static/randomly_trading/light/random-bonds-hhll.svg" />

### Cryptocurrencies
* 8 permutations

<img src="/static/randomly_trading/light/random-crypto-hhll.svg" />

## Review

The majority of the random trades decrease account equity across all asset classes tested (Forex, Indices, Bonds and Crypto). 

Theres are a few rare exceptions but I would be cautious to draw a strong conclusion from this as it would require the specific random trades. 

## AWS Cost Analysis

I've been using the Backtesting Engine throughout to deploy and test these trading strategies at scale on Amazon Web Services. 

The Backtesting Engine automates deployment and the output reporting for analysis. I've included a screenshot below of how the infrastructure looks on AWS when testing the Indices. 

<img src="/static/randomly_trading/aws-screenshot.png"   />

I’m using c7g.medium EC2 instances types for all my experiments. The c7g instances are Arm-based which have great performance at low cost. 

As an example, the Range of Fixed Targets for 10 Indices took 15 minutes for all the trading runs to finish, some sooner. If we round up, lets say they all took exactly 15 minutes it would cost roughly $2 USD to execute 250 permutations across 10 indicies over 9 years.

### AWS c7g Cost estimate

* 250 instances x 0.0361 USD On Demand hourly cost x 15 minutes (15/60) = **2.25625 USD**

### Improving Backtesting

Throughout this experiment I have been comparing the results with the performance in the live market. I've setup forward testing using Lambdas and DynamoDB on AWS to compare the differences. The AWS Lambdas inject the `IStrategy` and is a path to taking future strategies to the live market.

I discovered that the backtesting engine was too accurate. I originally put slippage at the start of every trade (-1 pip) however the close was exactly on the tick that met the stop loss (SL), which isn’t realistic. I found that there are typically delays between order and execution, so I have added a 1-second delay to the execution of a close trade request.

To give an example, a random trading strategy of +1 / -1 PIP previously looked like this:


```csharp
Date Time               PnL      Statement                      Profit  Direct  Open    Close
04/01/2023 08:44:57     393.65   Closed trade for EURUSD        -1.10   SELL    1.228   1.2286
04/01/2023 08:45:09     392.65   Closed trade for EURUSD        -1.00   BUY     1.229   1.2284
04/01/2023 08:46:49     393.65   Closed trade for EURUSD        1.00    SELL    1.228   1.2281
04/01/2023 08:47:20     392.65   Closed trade for EURUSD        -1.00   SELL    1.228   1.2283
04/01/2023 08:47:37     391.65   Closed trade for EURUSD        -1.00   BUY     1.228   1.2281
04/01/2023 08:49:08     390.65   Closed trade for EURUSD        -1.00   SELL    1.228   1.2283
04/01/2023 08:49:52     389.60   Closed trade for EURUSD        -1.05   SELL    1.228   1.2285
04/01/2023 08:51:31     388.60   Closed trade for EURUSD        -1.00   BUY     1.228   1.2283
04/01/2023 08:54:14     387.60   Closed trade for EURUSD        -1.00   BUY     1.228   1.2281
04/01/2023 08:54:49     388.60   Closed trade for EURUSD        1.00    BUY     1.228   1.2283
04/01/2023 08:56:05     387.55   Closed trade for EURUSD        -1.05   BUY     1.228   1.2281
04/01/2023 08:58:22     388.55   Closed trade for EURUSD        1.00    BUY     1.228   1.2284
```

After updating the backtesting engine a +1/-1 (TP/SL) looks like:

```csharp
Date Time               PnL      Statement                      Profit  Direct  Open    Close
05/01/2023 14:41:55     256.70   Closed trade for EURUSD        -1.05   BUY     1.227   1.2269
05/01/2023 14:44:14     257.50   Closed trade for EURUSD        0.80    BUY     1.227   1.2272
05/01/2023 14:44:56     256.65   Closed trade for EURUSD        -0.85   BUY     1.227   1.227
05/01/2023 14:46:48     257.40   Closed trade for EURUSD        0.75    SELL    1.227   1.2268
05/01/2023 14:46:59     256.50   Closed trade for EURUSD        -0.90   BUY     1.227   1.2267
05/01/2023 14:48:03     255.45   Closed trade for EURUSD        -1.05   SELL    1.227   1.2269
05/01/2023 14:49:16     256.90   Closed trade for EURUSD        1.45    SELL    1.227   1.2267
05/01/2023 14:50:32     257.85   Closed trade for EURUSD        0.95    BUY     1.227   1.2268
```

I'm going to research into how the markets move as I wonder if a reactive and adapting SL/TP would lead to better outcomes. I've started a new thread on [Market Analysis](/market_analysis/).

If you've got any feedback on this post please let me know <a href='mailto:ryan@mccaffers.com' target="_blank">ryan@mccaffers.com</a>

