---
title: "Futures, Options and Derivatives"
date: 2024-09-30T00:39:59+01:00
draft: false # Set 'false' to publish
tableOfContents: false # Enable/disable Table of Contents
description: 'My trading notes - learning about the common trading instruments'
categories:
  - financial markets
tags:
  - trading
  - forex
---

This blog entry contains my notes on these common trading instruments, as understood from this [Trading Riot's article](https://tradingriot.com/futures-options-and-derivatives) as well as around the web.

## Derivatives
A derivative is a contract that derives its value from an underlying asset -  which could be any asset actually - e.g: Bonds, Stocks, Crypto, commodities, etc.

One of the earliest mentions of derivatives is in ancient greek, where a derivative contract was used in trading olives.

There are two main ways in which derivatives are traded:
- Privately, over-the-counter (OTC)
- Exchange-traded derivatives (ETD)

Derivatives are used differently, depending on the market player:
- Large players use derivatives for hedging purposes
- Smaller/retail traders use derivatives for speculation and arbitrage.


## Futures
A futures contract is a legally-binding agreement to buy or sell a pre-determined amount of a  finnancial instrument at a specified price at a specified future date.
Although futures are now the most common financial instruments used in speculation, they have a history of being used as hedging instruments.

An example of a futures is Coffee-selling company A, which, in order to prevent themselves from being exposed to the fluctuating price of coffee, enter into an agreement to sell 1,000 50kg bags of coffee by Marcch 2030 at the price of $250 per bag.
Every future contract has its specific code, with the E-mini S&P being the most popular and traded futures contract globally. As at today (01.10.2024), the currently traded E-mini S&P has a code of **ESZ24**, where:
- *ES* represents the name of the contract
- *Z* stands for the month of expeiration (december in this case)
- *24* stands for the year of expiry

The expiration months are depicted as follows:
January   - F
Febraury  - G
March     - H
April     - J
May       - K
June      - M
July      - N
August    - Q
September - U
October   - V
November  - X
December  - Z

The most famous futures exchange is the Chicago Merchantile Exchange (CME) group. It offers index futures such as Dow Jones, E-mini S&P 500, Nasdaq as well as crypto futures like Bitcoin and ETH.

Commodities like Natural Gas, Crude Oil Gold, Silver are traded on New York Merchantile Exchange (NYMEX) and Commodities Exchange (COMEX) - which are both desginated contract markets, owned by the CME Group.
Outside the USA, there are other futures exchanges like EUREX, OSaka Exchange, ICE, MOEX, etc.


## Cryptocurrency Perpertual Futures (Perpertual Swaps)
Created in 1992, by Robert Shiller, to bring futures to illiquid assets, these contracts became the most popular future contracts after being introduced by Bitmex in 2016.
Like Index futures, Cryptpo futures settle in cash. The only difference, as the name implies, is that Crypto *Perpetual* futures do not have a set expiration date. As a result of this, traders of this futures pay/receive what is called a funding rate -  a fee that is paid in regular (originally 8-hourly) hourly windows.
- when the funding rate is +ve, long postion holders pay the short position holders.
- when the funding rate is -ve, short position holders pay long position holders.

The amount to be paid differs by position size and can be calculated using the formula:
#### funding fee =  position value * funding rate

Extremes of funding rates can be used to build a trading strategy, for instance, prolonged periods of high funding rates could be a signal that the market is being overbought/oversold (and could wear out soon).


## CFDs
CFD stands for Contract for Difference. It is another type of derivates that traders can trade. They are mainly offered by Forex brokers as a substitute for Futures contract. However, they are have a bad reputatation because of the model a lot of these brokers run CFDs on -  B-book model. In this model, they open positions against traders, this can essentially widen the spread of the CFDs.

### CFDs VS Futures
Just like Futures, CFDs are perpertual and do not expire. However, CFDs are largely unregulated as compared to futures, and are mostly discourgaed against. They are banned in the US and a number of other countries, but are offered by many European-regulated brokers.


## Options
Options give a holder the rights (not an obligation) to buy or sell an underlying asset at a specified price, on a specified date, at a sepcified associated premium.
For example, Mr A is owns a piece of land, and Mr B is looking to buy that same piece of land. Mr A issues Mr B an option to buy at at USD150,000, within a 6-month period, and at a premium of USD5,000.
In 6 months, the property value dropped to USD120,000 so Mr B  decides not to exercise his rights to buy it. He however, has to pay Mr the specified premium of USD5,000.
This however would have turnd out differently assuming the value of the property appreciated to USD200,000 and Mr B exercised his right to sell. He would have made a margin of USD45,000: (final price(200k) - strike price(150k) - premium(5k))
The above example can also be flipped for a sell scenario, same rules of premium apply.

The basic principle with Options is:
- As an options buyer, my risk is limited to the premium paid, but my reward is technically unbounded.
- As an options seller, my risk is technically unbounded, but my reward is limited to the premium received.

When trading Options, there are some key terms to keep in mind about an option:
- its moneyness
- its value

The **Moneyness** of an option describes the intrinsic value of the option's premium in the market.
When discussing an option's value, we are either referring to its **intrinsic value** or it's **extrinsic value**.


### Intrinsic value is the value of an option at expiration. 
An option's extrinsic value (also called it's time value) is a made up of a number of factors such as:
- the length of time until it's epiration (the longer the higher the extrinsic value).
- the dividends, volatility and risk-free interest rate of the underlying (asset).

### Options Trading Strategies
While trading some other derivatives involves going long or short, it's slightly different for options. With options, we trade **Calls** and **Puts**, and we can go long or short on them.

When I BUY A CALL, I'm BULLISH
When I SELL A CALL, I'm BEARISH
When I BUY A PUT, I'm BEARISH
When I SELL A PUT, I'm BULLISH

For both buy scenarios, my risk is limited to the premium paid and profit potential is unlimited, while for the sell scenario, my risk is unlimited and my profit potential is limited to the premium received.

When I buy a CALL, my profit potential applies when the market goes up, and when the market goes down when I buy a PUT.

Besides buying and selling CALLs and PUTs, with Options, I can also trade (buy & sell) volatility.

BUYing volatility simply implies that "I do not know how if the underlying would gain or lose value, but it would move more than the market expects", while SELLing volatility implies the inverse (moving less than the market expects).


