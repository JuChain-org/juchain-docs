# 4.4 Supply Dynamics and Vesting

To preserve long-term scarcity while supporting sustainable growth, JuChain employs a dual mechanism of controlled issuance and scheduled halvings.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': { 'primaryColor': '#ff0000'}}}%%
xychart-beta
    title "JU Token Emission Schedule"
    x-axis [Year 0, Year 4, Year 8, Year 12, Year 16]
    y-axis "Circulating Supply (millions)" [0, 50, 100, 150, 200]
    line [6.3, 120, 150, 170, 180]
```

_Figure: JU Token Supply Growth - Showing the effect of halving events on long-term supply_

## Genesis Supply and Circulation

* **Genesis Supply:** 210 million JU (100%) are minted at Token Generation Event (TGE)
* **Initial Circulation:** 3% (6.3 million) circulate immediately via the IEO
* **Locked Supply:** The remaining 97% are released over time via mining and ecosystem programs

## Daily Emission Curve

* **Initial Daily Output:** 72,000 JU
*   **Halving Schedule:** Every fourth year, the emission rate halves, following the formula:

    ```
    O_n = O_0/2^(n/4)
    ```

    Where:

    * O\_n is output in year n
    * O\_0 is initial output

## Projected Supply Growth

Under the halving model, circulating supply is expected to follow this trajectory:

* **Year 4:** \~120 million JU
* **Year 8:** \~150 million JU
* **Year 16+:** Asymptotically converges on 180 million (<90% of genesis supply)

This controlled emission schedule ensures that:

1. Token supply remains predictable
2. Early adopters are rewarded
3. Long-term supply is capped to preserve value
4. Emission rate adjusts to network maturity

## Vesting Schedules

### Foundation and Partner Vesting

Tokens accumulated by the JuChain Foundation and Partner Nodes are subject to:

* 12-month cliff (no tokens released)
* Linear vesting over 24 months after cliff period
* Public transparency through on-chain vesting contract

This vesting structure ensures alignment with long-term ecosystem success and prevents large amounts of tokens entering circulation at once.

## Liquidity Management

20% of net IEO proceeds are reserved for:

* Exchange-side liquidity
* Automated market-making pools

These provisions help stabilize early-stage price discovery and ensure healthy trading volumes from launch.
