+++
title = "Understanding How Indices Work"
date = 2025-01-03
description = "Understand the magic behind the MSCI World index used in the ETFs you blindly buy."
+++

An **index** is a theoretical portfolio designed to measure the performance of a specific segment of the market. It’s essentially a collection of securities representing a particular market, sector, or investment strategy.

Examples include the **S&P 500**, **MSCI World**, and **CAC 40**. These indices are managed by **index providers** like MSCI, S&P Dow Jones Indices, and Euronext.

# Constituents and Their Weights

## Market Capitalization-Weighted index

In a market-cap-weighted index, each company’s weight is proportional to its market value. The company’s market value being the company’s share value multiplied by the total number of shares. 

### **Advantages:**

- More representative of the market: Larger companies have a bigger impact on the index.
- Higher liquidity: Larger companies tend to be more liquid, making the index easier to track.

### **Disadvantages:**

- Concentration risk: A few large companies can dominate the index.
- Bubble risk: Overvalued companies may become even more overrepresented.
- Small-cap companies have a negligible impact on index performance.

## **Free Float-Adjusted Market Capitalization-Weighted Index**

Similar to the traditional market-cap-weighted index, but only publicly tradable shares are considered. Insiders’ shares and government-owned shares are excluded.

### **Advantages:**

- More representative of the market segments that are actually traded.
- Commonly used by ETFs to ensure alignment with tradable securities.

### **Disadvantages:**

- Companies with a significant portion of privately held or government shares are underrepresented.

### **Examples:**

- MSCI World, MSCI Emerging Markets, S&P 500, CAC 40.

## **Equal-Weighted Index**

In an equal-weighted index, every constituent has the same weight.

### **Advantages:**

- Increased diversification: Smaller companies have a larger impact.
- Reduced concentration risk: No single company or sector dominates.

### **Disadvantages:**

- Higher rebalancing costs: Frequent adjustments are needed as prices change.
- Smaller companies may be less liquid
- Higher transaction costs due to rebalancing.

# Rebalancing

If an index remains static, its constituents and their weights will become less representative of the market over time. This is because companies and sectors evolve, and market conditions change.

To address this, index providers periodically review and rebalance their indices.

- **Periodic Reviews:** Typically quarterly or semi-annually.
- **Exceptional Rebalancing:** Rarely, under extraordinary circumstances such as mergers, bankruptcies, or extreme market crashes.

### **Rebalancing Process**

- Providers evaluate constituents based on criteria, like:
    - Market capitalization.
    - Sector representation.
    - Liquidity.
    - Free float (number of tradable shares).
- Companies that no longer meet the criteria are removed, and new ones are added.
- Weights are recalculated and redistributed.

### **Types of Weight Adjustments**

1. **Between Reviews:** Weights change dynamically in real time due to stock price fluctuations. For example, if a company’s stock price rises, its market cap increases, and so does its weight in the index.
2. **During Reviews:** Weights are recalculated based on updated data, such as changes in market cap, free float, or index composition. These weights remain the baseline until the next review.

# **Impact of a Crash on an Index**

Consider the MSCI World index, where a few large U.S. tech companies make 20% of the index. If these companies valuation drop by 50%:

1. The total market cap of the index drops significantly (to 90% of its previous value in this example: 100% - (20% * 50%)).
2. The weight of these tech companies decreases, because their market cap in the index has been divided by 2.
3. As a result, their future impact on the index is reduced.

**Important Notes:**

- If no companies fall below the inclusion threshold, the constituent list remains unchanged, and only weights adjust.
- During the next review, companies with significantly reduced market caps might be removed, and new companies added, which could slightly alter the index value but not retroactively correct the crash’s impact.

# **Can You Buy an Index?**

An index is a mathematical construct and cannot be purchased directly. However, investors can buy ETFs that track an index.

### **Index vs. ETF**

- **Index:** A benchmark representing a market segment.
- **ETF:** An investable product designed to replicate the performance of the index.

If you’re into software engineering: think of the Index as the interface, and the ETFs as implementations of this interface.
