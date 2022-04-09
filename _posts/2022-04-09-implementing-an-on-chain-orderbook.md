---
layout: post
title: "Implementing an On-Chain Orderbook"
tags: 
    - markets
    - orderbooks
    - smart contracts
    - blockchains
    - DeFi
published: false
usemathjax: true
---
## Mental Model of a Market
Think of a market as a bulletin board where buyers and sellers can do one of two actions:

1. Make an order by posting on the bulletin board. For buyers, this means posting a bid price at which they are willing to buy. For sellers, this means posting an ask price at which they are willing to sell. A party which makes an order is known as a market maker or liquidity provider.
2. Take an order by removing a post from the bulletin board and fulfilling the corresponding order. A party which takes an order is known as a market taker.

One may ask whether market makers also have to post the quantity which they are willing to buy or sell. The answer is yes, but let's assume for simplicity that all orders posted on the bulletin board are for unit quantities. 

## Mental Model of an Orderbook
A market taker will buy at the lowest ask price and sell at the highest bid price posted on the bulletin board. An orderbook is simply a sorting  


## Implementing an On-Chain Orderbook