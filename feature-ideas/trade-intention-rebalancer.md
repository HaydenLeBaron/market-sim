# Feature: Trade Intention Rebalancer

**In a Nutshell:** 
- a user expresses their preferred ratios for their holdings (a "target").
- the system looks at their current state ("is") and their target state ("ought") and then creates a "trading intention* to bridge the two.
- a trade intention is processed into concrete buy/sell orders (in a double auction, for example) or whatever primitives are necessary according to the rules of a particular market/auction type.

