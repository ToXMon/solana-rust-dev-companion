# Slide deck extract: AMM theory (Turbin3 Builders Q3 2026)

Source: AMM theory.pdf
Method: pypdf text extraction; diagrams/images are not transcribed.

=== PAGE 1 ===
TURBIN3.ORGPAGE 1
Turbin3 
AMM Theory

=== PAGE 2 ===
TURBIN3.ORGPAGE 12
● Market makers and Evolution of Trading
● Constant Product Curve
● Arbitrage and Impermanent Loss
● Types of AMM
Agenda

=== PAGE 3 ===
TURBIN3.ORGPAGE 12
Definition: Market makers are entities that actively buy and sell securities or assets at 
publicly quoted prices, providing liquidity to the market.
Key Role: They facilitate trading by ensuring there's always someone willing to buy or sell, 
even when there's an imbalance in natural supply and demand
Inventory Management: They strategically manage their inventory to balance risk and 
potential profit.
How They Make Money:
a.Bid-Ask Spread: They profit from the difference between the price they buy at (bid) and 
the price they sell at (ask).
What are Market Makers

=== PAGE 4 ===
TURBIN3.ORGPAGE 12
● Great for new tokens without needing tons of liquidity
● Games and lower liquidity tokens
The AMM and its purpose


=== PAGE 5 ===
TURBIN3.ORGPAGE 12
1.Individuals or entities who deposit their assets into liquidity pools.
2.LPs earn a share of trading fees as a reward for providing liquidity.
3.Required to deposit two tokens
a.X token
b.Y token
4.Received the Lp token represents their portion of the pool, nominal not 
notional 
What is an LP token

=== PAGE 6 ===
TURBIN3.ORGPAGE 12
CPMM K=XY
 1.Example: When someone sells token A, 
the AMM buys these tokens and givens 
them Token B. 
2.This lowers token B and increases the A 
supply. Increasing price B and lowering 
price A price
3.This is a spread akin to MM 


=== PAGE 7 ===
TURBIN3.ORGPAGE 12
Initial State: The AMM has reserves of Token X 
and Token Y, and a constant k in the xy = k formula.  
● For simplicity, let's say k is 600, and the pool 
initially has 20 units of Token X and 30 units of 
Token Y. 
SWAP Order: Trade 1:
Action: SWAP 5 X
Calculation:
X2 = 20 + 5 = 25
Y2 = 600 / 25 = 24
Y change = 30 - 24 = 6
Result: 
Received 6 Y for 5 X
Sell Order Example Rebalancing & New State: The AMM needs to 
maintain the xy = k invariant to keep k = 600, the AMM 
Example 2:
X: 25, Y: 24, K = 600
Action: SWAP 5 X
Calculation:
X2 = 25 + 5 = 30
Y2 = 600 / 30 = 20
Y change = 24 - 20 = 4
Result: Received 4 Y for 5 X

=== PAGE 8 ===
TURBIN3.ORGPAGE 12
Price Difference: The CPMM now values A at 0.59 B, but other markets 
might still value it closer to the original price of 1 B per A
Buy Low, Sell High: Arbitrageurs can buy A cheaply from the CPMM at 0.59 
B and immediately sell it on another exchange for closer to 1 B, making a 
profit on the difference
Important Points to Consider:
1.Fees
2.Gas
AMM Arb

=== PAGE 9 ===
TURBIN3.ORGPAGE 12
Impermanent Loss: The constant rebalancing to maintain k can lead to LPs 
having a different ratio of assets than they initially deposited, potentially 
resulting in a loss compared to simply holding the tokens.
Fees as Compensation: LPs earn trading fees on each swap, which helps 
offset the risk of impermanent loss.
IL or Divergent Loss

=== PAGE 10 ===
TURBIN3.ORGPAGE 12
Two Main Types of Order Flow:
1.Informed: Driven by traders with privileged information, potentially 
impacting market prices.
2.Uninformed: Based on publicly available information and sentiment, less 
likely to significantly impact prices.
Two Types of Uninformed Flow:
3.Non-Toxic: Patient, deliberate trades by long-term investors, contributing 
to a healthy market.
4.Toxic: Aggressive, high-frequency trading that creates artificial volatility 
and can harm market quality.
Understanding Order Flow in Trading

=== PAGE 11 ===
TURBIN3.ORGPAGE 16
Q&A