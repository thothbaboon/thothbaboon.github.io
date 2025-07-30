+++
title = "Everything You Need to Know About How ETFs Work"
date = 2025-01-03
description = "Understand the magic behind the MSCI world ETF you blindly buy."
+++

An **Exchange-Traded Fund (ETF)** is a type of investment fund that owns financial assets, such as stocks, bonds, or other securities, and is traded on a stock exchange, just like regular stocks.

Most ETFs aim to replicate the performance of a specific index.

For example, **iShares Core MSCI World UCITS** tracks the performance of the MSCI World Index.

In this setup: 

- MSCI is the index provider, responsible for designing, maintaining, and updating the index. The index is essentially a mathematical model representing the performance of a specific market.
- iShares (BlackRock) is the ETF issuer, which constructs the ETF to closely mirror the index's performance. iShares pays licensing fees to MSCI for the right to use their index.

 Let’s dive deeper into the mechanics to fully understand how ETFs operate under the hood.

# How ETFs Track Indices

## Physical Replication

Physical replication involves the ETF directly purchasing and holding the actual assets that make up the underlying index.

### Full replication

The ETF holds all the assets in the same proportions (weights) as they appear in the index.

**Advantages**:

- Provides a highly accurate representation of the index.
- Results in low tracking error, as the ETF closely mirrors the index's performance.

**Disadvantages**:

- Higher transaction costs, especially for indices with many constituents.
- Some assets may lack sufficient liquidity, making them harder or more expensive to trade.
- Fractional shares might be required for assets with very small weights in the index, adding complexity.

### Sampling

The ETF holds only a representative sample of the index’s assets. These assets are chosen based on factors such as liquidity or sector.

**Advantages**:

- Cost-efficient, as fewer transactions are required.
- Works well when some index constituents are illiquid (e.g., emerging markets or small-cap stocks).

**Disadvantages**:

- Higher tracking error compared to full replication, as the ETF doesn't hold all the assets in the index.

## Synthetic Replication

Synthetic replication uses financial derivatives instead of directly holding the underlying assets.

Usually, the ETF issuer enters into a **swap agreement** with a counterparty, such as an investment bank.

The counterparty promises the swap will deliver the index’s performance and is responsible for ensuring the ETF gets the correct performance

**Advantages**:

- Enables exposure to markets or assets that are difficult to access directly.
- Can be more cost-effective than physical replication.

**Disadvantages**:

- If the counterparty fails to honor the swap agreement, the ETF may not achieve its intended performance.
- Synthetic replication is harder for investors to understand: “What exactly am I buying?”

# Understanding ETF Shares and Price

ETF shares represent something fundamentally different from stock shares.

## How Stock Shares Work

The valuation of a publicly traded company is determined by its **market capitalization**, which is calculated as:

`Market Cap = Stock Price * Number of Shares`

The stock price is driven by **supply and demand** in the market. When more buyers than sellers want a stock, the price tends to rise; when sellers outnumber buyers, the price falls.

If the number of shares changes (e.g., due to a stock split, share buyback, or issuance of new shares), the stock price adjusts accordingly to ensure the market capitalization remains consistent, assuming no other factors affect the valuation.

## How ETF Shares Work

For ETFs, the process is different. The **number of ETF shares** is not fixed. It adjusts based on demand.

When demand increases, new ETF shares are created, which also requires increasing the holdings of the ETF’s underlying assets in proportion.

Conversely, when demand decreases, ETF shares are redeemed, and the underlying assets are sold.

ETF shares are traded on stock exchanges, just like individual stocks. The prices of stocks are driven purely by supply and demand from buyers and sellers in the market.

In contrast, while ETFs are also influenced by market forces, their prices are not entirely dictated by buyers and sellers. ETFs are designed to closely track the performance of an underlying index.

How do the prices of ETFs follow the index instead of being purely market-driven?

This will become clear when we will explore Arbitrage later. 

But first, we need to understand the **creation and redemption process**.

For now, keep in mind that ETFs have two key prices:

### Net Asset Value (NAV)

The **NAV** represents the value of the ETF based on its underlying assets.

`NAV = (Asset + Cash - Liability) /  Number of Shares`

<aside>
💡

Aside from the underlying assets of the index, an ETF can also hold a small portion of its value in cash and liabilities. Dividends are a notable example, which we will discuss in more detail later.

</aside>

### Market Price

The **Market Price** is the price at which the ETF is traded on stock exchanges. This price is determined by buyers and sellers in the market.

However, due to the ETF’s structure and the role of arbitrage, the market price is kept close to the NAV.

## Market Cap Dynamics: Stocks vs ETFs

### For Stocks

The market capitalization of a company is calculated as:

`Market Cap = Stock Price * Number of Shares`

When more investors want to buy a stock, the company cannot simply create new shares to meet demand. Issuing new shares is a **corporate process** that involves strategic decisions and regulatory oversight.

Instead, what changes is the **stock price**, driven by supply and demand in the market.

As demand pushes the stock price higher, the market capitalization increases. This reflects the market's perception of the company's increased value.

### For ETFs

An ETF must closely track the performance of its underlying index. For this reason, the ETF price must stay close to the NAV.

To maintain this alignment, the supply of ETF shares adjusts rather than the price moving significantly away from the NAV.

When demand for the ETF rises:

- New ETF shares are **created** through the creation process.
- The ETF’s market cap increases as more shares are issued, and more underlying assets are added to the ETF portfolio.

However, **the NAV remains unchanged**, because the increase in the number of shares corresponds proportionally to the increase in the ETF’s total assets.

## Dividends

Some companies pay dividends periodically in cash, typically in the currency of their stock. For an ETF like the MSCI World ETF, this means receiving dividends from potentially hundreds of constituent stocks.

ETFs handle dividends in two primary ways, depending on their type:

If the ETF is Distributing:

- When dividends are received, they are recorded as **cash assets** on the ETF’s balance sheet, temporarily contributing to the ETF’s NAV.
- These cash assets are then distributed to ETF shareholders as cash payments. Once the dividends are scheduled for payout, they are recorded as a **liability** until paid.

If the ETF is Accumulating:

- The converted dividends temporarily become part of the ETF’s **cash holdings**, increasing the NAV.
- The ETF uses this cash to purchase additional shares of the underlying assets.

Note that dividends received in foreign currencies are first converted into the ETF's base currency. Any foreign exchange fees reduce the amount available for reinvestment or distribution.

# How ETF shares are created: The Process of Creation and Redemption

The process of **creation and redemption** adjusts the supply of ETF shares to match market demand. 

## The Creation Process

When demand for an ETF increases, new shares are created.

ETF issuers work with **Authorized Participants (APs)** - large financial institutions with the capability to operate cost-effectively in the markets.

Every day, the ETF issuer publish a **Portfolio Composition File (PCF)** with the constituents of the index and their respective weights.

The AP assembles a basket of assets that mirrors the index's composition and proportions, and delivers it to the ETF issuer.

In return, the ETF issuer provides the AP with a **creation unit:** a block of ETF shares. The standard number of shares by creation unit is 50,000).

If the **ETF NAV** is $10 and the **creation unit** is 50,000 shares, the AP must provide a basket of assets worth $500,000 (€10 × 50,000).

The AP is free to sell these newly created ETF shares on the a stock exchange.

The AP and ETF issuer interact on what’s called the primary market. This is where creation and redemption happen.

The secondary market is where ETF shares are traded among investors on the stock exchanges.

## The Redemption Process

When demand for an ETF decreases, shares are redeemed. This is essentially the reverse of the creation process.

AP buys ETF shares on secondary market until it has enough to form a creation unit.

It then delivers the creation unit to the ETF issuer.

In exchange of this creation unit, the ETF issuer provides the AP with the equivalent value in underlying assets form the ETF portfolio.

# ETF Price Arbitrage

AP uses the process of creation and redemption to keep the market price close to the NAV.

The closer the price is to the NAV, and the closer the ETF’s underlying assets are to the index’s underlying assets, the better the tracking. Therefore, arbitrage is a essential.

### ETF Market Price > NAV (Trading at a Premium)

If the market price rise, it means there are more buyers than sellers for the ETF, so more demand.

- AP purchases the underlying assets that make up the ETF's tracked index in the same proportions.
- AP delivers these assets to the ETF issuer in exchange for new ETF shares.
- AP sells the newly created ETF shares on the open market at the higher market price.

**Effect on the Market**

- The **increased supply** of ETF shares on the market (as the AP sells the newly created shares) puts downward pressure on the ETF market price.
- Simultaneously, the AP's purchase of the underlying assets increases demand for those assets, potentially raising their prices, which in turn increases the NAV.

The ETF market price moves downward, and the NAV moves upward, reducing the premium.

### ETF Market Price < NAV (Trading at a Discount)

If the market price drop, it means there are more sellers than buyers for the ETF, so less demand.

- AP buys ETF shares on the open market at the lower price.
- AP redeems these ETF shares with the issuer in exchange for the underlying assets.
- AP sells the underlying assets on the market at their market value.

**Effect on the Market**:

- The **reduced supply** of ETF shares on the market (as the AP redeems shares) puts upward pressure on the ETF price.
- Simultaneously, the AP's sale of the underlying assets increases their supply, potentially lowering their prices, which in turn decreases the NAV.

The ETF market price moves upward, and the NAV moves downward, reducing the discount.

# How the Different Actors Involved Make Money

Several key players are involved in the ETF ecosystem, and each earns revenue for their role. Here’s a breakdown of how they make money.

## Authorized Participants (APs)

### Role

- Handle the creation and redemption process which helps maintain ETF liquidity.
- Uses arbitrage to keep the market price aligned with the NAV.

### How they make profit

- From arbitrage, exploiting the discrepancies between the ETF’s market price and its NAV. This is risk-free revenue.
- When executing the creation/redemption process, APs may profit from the bid-ask spreads of the underlying assets.

## Market Makers

### Role

- Provide liquidity in the secondary market, ensuring ETFs can be bought or sold at any time.

### How they make profit

- From the Bid ask spreads, as Market makers are continuously providing buy and sell quotes.

<aside>
💡

Some APs can also be Market Makers. But APs operate primarily on the primary market, whereas market makers focus only on the secondary market.

</aside>

## ETF Issuer

### Role

- Create and manage the ETF: underlying portfolio, compliance, day-to-day operations, ..

### How they make profit

- From the management fees, that are detailed in the description of the ETF.
    
    <aside>
    💡
    
    a 0.5% management fee means that for every €1,000 invested, the issuer earns €5 annually.
    
    </aside>
    
- From lending some underlying assets to other institutions (e.g., for short selling). The ETF issuer receive earn interest on these loans.
    
    <aside>
    💡
    
    A portion of this revenue may be returned to investors. If so, it’ll be specified in the ETF description.
    
    </aside>
    

## Stock Exchanges

### Role

- Provide a platform for ETF trading in the secondary market.

### How they make profit

- Every time an ETF is bought or sold on the exchange, the stock exchange earns a **transaction fee** from the trade.
- ETF issuers pay stock exchanges a **listing fee**, to list their ETFs, ensuring their products are available to investors.

## Index Providers

### Role

- Design and maintain the indexes

### How they make profit

- ETF issuers pay fees to index providers for the right to use their indexes. For example, MSCI charges licensing fees to ETF issuers who want to track the MSCI World Index.

## Investors

### Role

- Invest into ETFs

### How they make profit

- As the ETF’s price rises over time, investors benefit from price gains.

## Custodians

### Role

- ETF issuer often charge a custodians to safeguard the underlying assets.

### How they make profit

- Custodians charge the ETF issuer fees

## Regulatory bodies

### Role

- Ensure compliance with laws and regulations.
- Protect investors by enforcing transparency and fair trading practices.

### How they make profit

- Charge ETF issuers for registering funds and filing periodic reports

# Conclusion

In this article, we’ve covered 80% of what you need to know about ETFs. We focused on index ETFs, as they are the most commonly purchased, especially for investing in indices like the MSCI World, USA, or emerging markets.

If this article has raised more questions in your mind, I’m sure you now have the keys to explore the remaining 20% on your own !
