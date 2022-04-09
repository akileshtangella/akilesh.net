---
layout: post
title: "How Dai Algorithmically Stays Pegged to the US Dollar"
tags: 
    - stablecoins
    - blockchains
    - DeFi
published: false
usemathjax: true
---
Dai is an Ethereum based ERC20 token. The protocol behind the Dai is known as the Maker Protocol. The purpose of Dai is that at all points in time, its price should be equal to 1 US Dollar (USD). This is known as being pegged to the dollar. The price of a conventional ERC20 token is determined by supply and demand on the open market. This leads to unpredictable price volatility. So how does the Maker Protocol, despite the Dai being traded on the open market, kill this price volatility, and instead stay pegged to the USD. In this article, we answer these questions.

## Creating Dai via Collateralized Debt Positions
Before we explore price stability 

## Price Stability Mechanism #1: Arbitrage
Suppose a token is traded on two exchanges, $$A$$ and $$B$$. Suppose for simplicity, there are no transaction fees. Let the token's price on exchange $$A$$ be $$P_A$$ and its price on exchange $$B$$ be $$P_B$$. We claim that the $$P_A = P_B$$. Suppose for the sake of contradiction that this is not true and $$P_A < P_B$$. An opportunistic trader will buy tokens on exchange $$A$$ and sell them on $$B$$, thus making a profit. This will occur until the $$P_A$$ and $$P_B$$ converge. 

Suppose we've locked some ETH into a CDP and the price of Dai drops below 1 USD. We will immediately buy some Dai and use it to pay down our debt in the CDP. This action removes Dai supply from the open market, thus increasing its price. On the other hand, suppose the price of DAI goes above 1 USD. Then we will open a CDP and mint some DAI, since the value of the debt in the CDP 

## Price Stability Mechanism #2: 

## Price Oracle



