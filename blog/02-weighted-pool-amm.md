# The Weighted Pool AMM: One Pool to Price Your Entire Economy

You have eight commodities in your game — Wood, Iron, Stone, Food, Gold, Leather, Herbs, Gems — and you need agents to be able to trade any one for any other. With a traditional order book (the CDA from our previous post), you'd need a separate market for each trading pair. That's `8 * 7 / 2 = 28` order books, each requiring its own liquidity, its own bid-ask spread, its own set of active traders.

Most of those 28 markets will be ghost towns. Nobody is sitting around posting bids on the Herbs/Leather exchange.

A **Weighted Pool AMM** solves this in one stroke: dump all eight assets into a single pool, define target weight ratios, and let agents swap any asset for any other against that shared reserve. The pool does the pricing automatically, maintains coherent cross-rates between all assets, and gets more accurate over time as agents trade through it.

This is how Balancer works in DeFi. For your game economy, it's arguably more powerful — because you don't have external arbitrageurs to rely on. The pool *is* your price discovery mechanism.

---

## The Core Idea

A standard constant-product AMM (like Uniswap) holds two assets and enforces the invariant `x * y = k`. The Weighted Pool generalizes this to *N* assets with arbitrary weights:

```
(x₁^w₁) * (x₂^w₂) * ... * (xₙ^wₙ) = k
```

Where:
- `xᵢ` is the reserve quantity of asset *i* in the pool
- `wᵢ` is the weight of asset *i* (all weights sum to 1)
- `k` is a constant that only changes when liquidity is added or removed

The weights express the target value composition of the pool. A pool weighted 40% Gold / 20% Wood / 20% Iron / 10% Food / 10% Stone is saying: "At equilibrium, 40% of the total pool value should be Gold."

The price of any asset relative to any other falls directly out of the math:

```
Price of A in terms of B = (xB / wB) / (xA / wA)
```

When a trader swaps Wood for Iron, the Wood reserve goes up, the Iron reserve goes down, and the price of Wood (in Iron terms) drops while the price of Iron rises. The invariant ensures the pool always has *something* to offer — it can never be fully drained of any asset.

---

## Why This Matters: Coherent Cross-Pricing for Free

Here's the property that makes weighted pools uniquely powerful for a game economy.

Suppose an agent trades a large quantity of Wood for Iron. That single trade changes the Wood/Iron price. But because all assets live in the same pool governed by the same invariant, it **simultaneously and automatically** adjusts:

- Wood/Stone price
- Wood/Food price
- Iron/Stone price
- Iron/Food price
- ... every other pair involving Wood or Iron

In an order-book world, a big Wood/Iron trade tells you nothing about the Wood/Stone price unless someone happens to be trading on that pair too. In a weighted pool, every trade implicitly reprices the entire economy.

This means the pool maintains a **coherent, arbitrage-free web of exchange rates** across all *N* assets at all times. You don't need 28 liquid markets. You need one pool and enough total trade volume to push reserves toward their fair values.

---

## Solving the Double Coincidence of Wants

Remember the fundamental problem of barter: Alice has Wood and wants Iron, but Bob has Iron and wants Food, not Wood. No trade.

A weighted pool is a **universal counterparty**. Alice doesn't need to find someone who specifically wants Wood and has Iron. She swaps Wood into the pool and takes Iron out. Bob swaps Iron into the pool and takes Food out. The pool mediates every exchange. The double coincidence of wants — the ancient constraint that made barter scale so poorly — is eliminated for the entire economy in one mechanism.

---

## Walkthrough: A Five-Asset Game Economy Pool

Let's set up a pool for a game with five resources, weighted by how "valuable" we want them to be at equilibrium:

| Asset | Weight | Initial Reserve | Initial Price (in Gold) |
|---|---|---|---|
| Gold | 30% | 1,000 | 1.00 (numeraire) |
| Iron | 25% | 5,000 | 0.167 |
| Wood | 20% | 10,000 | 0.067 |
| Food | 15% | 15,000 | 0.033 |
| Stone | 10% | 20,000 | 0.017 |

Now suppose the Blacksmith agent needs Iron to forge swords. It has surplus Wood. It submits a swap: **500 Wood for Iron**.

The pool calculates how much Iron to give out based on the invariant. After the swap:

- Wood reserve increases (10,000 → 10,500): Wood becomes slightly cheaper.
- Iron reserve decreases (5,000 → ~4,762): Iron becomes slightly more expensive.
- **Every other cross-rate adjusts**: Wood/Food, Iron/Stone, Wood/Gold — all updated automatically.

The Blacksmith got its Iron. The pool got Wood. No counterparty was needed. And the entire price grid of the economy shifted marginally to reflect the fact that someone just signaled "Wood is less scarce than Iron."

---

## Advanced Strategies the Pool Enables

### Asymmetric Exposure for Liquidity Providers

An agent that is bullish on Iron (maybe it knows a war is coming and Iron demand will spike) can provide liquidity to an 80% Iron / 20% Gold pool. Because the invariant heavily weights Iron, the LP's portfolio stays mostly in Iron even as prices fluctuate. It earns trading fees while maintaining its Iron-heavy position. This is a direct analogue to how Balancer LPs manage impermanent loss.

### Self-Rebalancing Treasuries

A merchant NPC holds a portfolio of all five resources. Instead of actively rebalancing — "I have too much Wood, sell some for Iron" — it deposits its portfolio into the pool. When Wood spikes in value, traders are mathematically incentivized to pull Wood out of the pool (buying it cheap) and deposit other assets, automatically rebalancing the merchant's holdings back to target weights. The merchant earns fees for this service. It's a passive index fund run by a math formula.

### Liquidity Bootstrapping (Dynamic Dutch Auction)

You introduce a new resource — say, **Mithril** — and need to discover its fair price. Set up a pool initially weighted 95% Mithril / 5% Gold, then programmatically shift the weights toward 50/50 over time. The shifting weights create continuous downward price pressure on Mithril (like a Dutch auction that slowly drops the price), and agents step in to buy when the price hits their valuation. The result: fair price discovery for a brand-new asset without anyone having to guess the right starting price.

---

## Implementation Considerations

**The invariant computation.** You're computing products of values raised to fractional powers. Use logarithms to avoid overflow: `ln(k) = w₁*ln(x₁) + w₂*ln(x₂) + ...`. This turns the multiplicative invariant into a sum, which is numerically stable and fast.

**Swap output calculation.** Given input amount `Δxᵢ` of asset *i*, the output amount `Δxⱼ` of asset *j* is:

```
Δxⱼ = xⱼ * (1 - (xᵢ / (xᵢ + Δxᵢ))^(wᵢ/wⱼ))
```

This is a closed-form expression — no iterative solving required. Fast enough to run on every game tick.

**Fees.** Charge a small percentage fee on every swap (e.g., 0.3%). This serves two purposes: it rewards liquidity providers, and it dampens noise trading. Fees accrue inside the pool, increasing `k` over time.

**Slippage is a feature, not a bug.** Large trades move the price significantly. An agent trying to buy half the pool's Iron in one transaction will pay an astronomical effective price. This is the pool's natural defense against being drained. It also signals to agents: "If you need a lot of Iron, you'll move the market. Consider breaking your trade into smaller pieces." This creates realistic market dynamics.

**Weight selection is economic design.** The weights you choose reflect your game's intended value hierarchy. If Gold should be 10x more valuable per unit than Wood, your weights and initial reserves should reflect that. Get this wrong and agents will immediately drain the mispriced asset. Get it right and the pool becomes a stable backbone for your economy.

---

## When to Use a Weighted Pool in Your Game

Use a weighted pool as the **economy-wide pricing backbone** when you have multiple fungible commodities and want coherent relative pricing without maintaining dozens of individual order books.

It is especially powerful when:
- You have many asset types and few agents (thin markets where order books would be illiquid).
- You want automatic price discovery for new resources without bootstrapping individual markets.
- You need a universal counterparty so agents can always trade, even when no other agent wants the opposite side right now.

It is *not* ideal when you need precise price-time priority (use a CDA), when you're auctioning unique items, or when the assets in question have strong complementarities that demand bundle pricing (use a combinatorial auction).

For many game economies, the best design is a hybrid: a weighted pool as the always-available baseline market, with CDAs layered on top for the highest-volume trading pairs where agents want tighter spreads and more control over execution.
