# My reactions and questions to blogs 01-04
(
  originally written in the margins of 02-weighted-pool-amm.dm
)
- could we consider each agent's inventory to be a "Weighted pool AMM"--where each agent is a "liquidity provider" (you could barter with anyone). Most agents don't want most things and so their equilibrium weights for most assets are 0%. The weights represent the relative proportions of a thing they want to hold. At any given time tick, we can consider the weights to be constants. However, over time--as agents get hungrier, greedier, lust for power, the weights of their relative desires for certain assets will change. When an agent participates in a market event, we can consider their weights can be used to calculate the MR[S/T] (Minimum Rate of Substitution/Transformation) for that agent (multi-dimensional vector). We can then use this to determine how much surplus of each thing they have--to contribute to the market's pool--and then they can take turns heuristic swaps with the pool until they get the balance of what they want.

Because "constant" (on any given turn) weights are derived from the relative subjective values of each agent, the relative values need not be "correct" in any objective sense--they just need to be consistent with the agent's current desires. Therefore, market value is an emergent property of agent desires.



```md
// gemini
When you designate one of the goods as a currency or "numéraire" good (let's call it $m$ for money), and you have $N$ other commodities, your utility function is $U(x_1, \dots, x_N, m)$.
By doing this, you instantly collapse the complex web of $\frac{N(N-1)}{2}$ direct good-to-good trade-offs into a clean $N$-dimensional vector of MRS values against money:
(MRS1,m,MRS2,m,…,MRSN,m)=(MU1MUm,MU2MUm,…,MUNMUm)(MRS1,m​,MRS2,m​,…,MRSN,m​)=(MUm​MU1​​,MUm​MU2​​,…,MUm​MUN​​)
Why this is so powerful
1. It perfectly matches the price vector
In a market economy, consumers don't have to think about the exchange rate between apples and haircuts. They only have to look at the price of apples in money ($P_1$) and the price of haircuts in money ($P_2$).
At the optimal choice (consumer equilibrium), a person will buy goods until their internal, subjective MRS for each good against money equals the market price of that good:
MRSi,m=PiMRSi,m​=Pi​
(Assuming the price of money $P_m$ is exactly 1).
So, that $N$-dimensional vector of MRS values against currency is exactly the vector of prices the consumer is willing to pay.
2. It mathematically routes all other pairs
Even though you only explicitly compute the $N$ pairs against money, you haven't lost the $\frac{N(N-1)}{2}$ other combinations. They are implicitly locked in by the math. If you ever did want to know the MRS between good 1 and good 2, you just divide their MRS against money:
MRS1,2=MRS1,mMRS2,m=P1P2MRS1,2​=MRS2,m​MRS1,m​​=P2​P1​​

The Numéraire
In microeconomics, the good against which all other MRS values are evaluated is called the numéraire good. While it is often literal money (currency), economic models sometimes just pick one random commodity (like "gold" or "leisure" or "a composite of all other goods") and set its price to 1.
By using a numéraire, you formally reduce the complexity of the economy from an explosion of pairwise combinations down to a simple $N$-dimensional price system, which is exactly why monetary economies are vastly more efficient for information processing than barter economies.
```

So I wonder:
- could you generate each agent's MRS for each good against a numéraire good (gold) and then from there we know how they will make bids/asks in CDAs or pairwise CPMMs (settled in gold)?
- you could also generate the MRS for each good against another to know under what conditions they would swap in a weighted pool AMM.
  - but how to rationally set the weights?

