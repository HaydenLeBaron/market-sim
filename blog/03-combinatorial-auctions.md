# Combinatorial Auctions: When the Bundle Is Worth More Than the Sum of Its Parts

Your Blacksmith agent needs 10 Iron *and* 5 Wood *and* 2 Leather to craft a Steel Sword. It goes to the Iron market, bids aggressively, and wins — paying a premium because it really needs that Iron. Then it goes to the Wood market and gets outbid. Now it's stuck with expensive Iron it can't use without the Wood. It overpaid for half a recipe.

This is the **exposure problem**, and it is the central failure mode when agents with complementary needs are forced to buy items one at a time in separate markets.

A **Combinatorial Auction** fixes this by letting agents bid on *bundles* of items. The Blacksmith doesn't bid on Iron, Wood, and Leather separately — it submits a single bid: "$200 for {10 Iron, 5 Wood, 2 Leather}, or nothing." It either gets everything it needs to craft the sword, or it gets nothing and keeps its money. No exposure.

---

## The Exposure Problem, Concretely

Consider why this matters in a game economy with crafting. Agents don't just want raw materials for their own sake — they want *combinations* that unlock crafting recipes, defensive structures, or military units.

| Agent | Needs | Bundle Value | Value of Any Subset |
|---|---|---|---|
| Blacksmith | 10 Iron + 5 Wood | $200 (craft a sword) | ~$20 (raw resale) |
| Builder | 20 Stone + 10 Wood | $150 (build a wall) | ~$30 (raw resale) |
| Alchemist | 5 Herbs + 3 Gems | $300 (brew a potion) | ~$15 (raw resale) |

The Blacksmith values {Iron + Wood} at $200 because together they produce a sword. Individually, the Iron is worth maybe $12 and the Wood maybe $8 on the commodity market. The synergy — the crafting recipe — creates a value of $200 that only exists when *both* items are secured together.

If you force the Blacksmith to bid in two separate markets, rational behavior gets complicated fast. It must overbid on one item to ensure it gets it, knowing that if it fails on the second, the first purchase was a waste. Agents end up either bidding timidly (and never completing recipes) or bidding recklessly (and frequently getting stuck with useless partial bundles). Neither outcome is efficient.

---

## How It Works

### Bidding Phase

Agents submit bids on subsets of the available items. Each bid specifies:
1. The **bundle**: a set of items (e.g., {10 Iron, 5 Wood, 2 Leather})
2. The **price**: what the agent will pay for that exact bundle

An agent can submit multiple bids on different bundles. "I'll pay $200 for {Iron + Wood + Leather}, OR $120 for just {Iron + Wood}, OR $80 for just {Iron}." This is called an **XOR bid** — the agent wins at most one of its bids.

### The Winner Determination Problem (WDP)

Once all bids are in, the auctioneer must decide which combination of non-overlapping bids to accept in order to maximize total revenue (or total social welfare — the sum of winning bidders' valuations).

This is where combinatorial auctions get computationally serious. The auctioneer is solving an optimization problem:

> Maximize the total value of accepted bids, subject to the constraint that no item is allocated to more than one winning bid.

This is equivalent to the **Set Packing problem**, which is **NP-hard**. With *N* items, there are `2^N - 1` possible bundles. For 20 items, that's over a million possible bundles. For 30, it's over a billion.

In practice, this is solved with **integer linear programming (ILP)**. You define a binary variable for each bid (accept or reject), maximize the sum of accepted bid values, and add constraints ensuring each item appears in at most one accepted bid. Modern ILP solvers (CPLEX, Gurobi, or open-source alternatives like CBC) can handle auctions with thousands of items and bids, though solve time can be unpredictable.

### Clearing

The solver outputs the winning set of bids. Each winner pays their bid price (in a first-price variant) or a calculated price based on the VCG mechanism (in a second-price variant — more on that below). Items are allocated, and the auction is complete.

---

## Walkthrough: Crafting Recipe Auction

The kingdom is auctioning off its treasury of raw materials:

**Available items:** 20 Iron, 15 Wood, 10 Leather, 8 Herbs, 5 Gems

**Bids received:**

| Agent | Bundle | Bid |
|---|---|---|
| Blacksmith | {10 Iron, 5 Wood, 2 Leather} | $200 |
| Builder | {15 Wood, 5 Iron} | $140 |
| Alchemist | {8 Herbs, 5 Gems} | $300 |
| Weaponsmith | {15 Iron, 10 Leather} | $250 |
| Carpenter | {10 Wood, 3 Leather} | $100 |
| Jeweler | {5 Gems} | $180 |

The auctioneer needs to find the revenue-maximizing allocation where no item is double-assigned.

**Option A:** Blacksmith + Alchemist + Carpenter
- Uses: {10 Iron, 5 Wood, 2 Leather} + {8 Herbs, 5 Gems} + {10 Wood, 3 Leather}
- Check: 10 Iron (of 20 avail), 15 Wood (of 15 avail), 5 Leather (of 10 avail), 8 Herbs (of 8 avail), 5 Gems (of 5 avail). Feasible.
- Revenue: $200 + $300 + $100 = **$600**

**Option B:** Weaponsmith + Builder + Alchemist
- Uses: {15 Iron, 10 Leather} + {15 Wood, 5 Iron} + {8 Herbs, 5 Gems}
- Check: 20 Iron (of 20 avail), 15 Wood (of 15 avail), 10 Leather (of 10 avail). Feasible.
- Revenue: $250 + $140 + $300 = **$690**

**Option C:** Weaponsmith + Builder + Jeweler
- Revenue: $250 + $140 + $180 = $570. Lower.

**Winner: Option B** — the Weaponsmith, Builder, and Alchemist each get their full bundles. No agent is left holding half a recipe. The Blacksmith loses out entirely, but that's better than winning Iron and failing to get Wood.

---

## The VCG Mechanism: Truthful Combinatorial Auctions

The **Vickrey-Clarke-Groves (VCG)** mechanism is the theoretical gold standard for combinatorial auctions. It's the multi-item generalization of the second-price (Vickrey) auction.

Under VCG:
- The auctioneer solves the WDP to find the welfare-maximizing allocation.
- Each winner pays the **externality** they impose on others: the difference between (a) the total value other bidders would have achieved without this winner, and (b) the total value other bidders actually achieved.

The magic property: **truthful bidding is a dominant strategy**. Every agent's best move is to bid exactly what each bundle is worth to them. No shading, no strategic inflation, no game theory needed on the bidder side.

The catch: VCG requires solving the WDP multiple times (once for the full auction, then once more for each winner to compute their payment). It can also yield surprisingly low revenue in edge cases. But for a game simulator where you control the solver and value truthfulness, VCG is a natural choice.

---

## Implementation Considerations

**The WDP is your bottleneck.** Everything else in a combinatorial auction is straightforward — collecting bids, allocating items, charging payments. The hard part is the optimization. For small auctions (under ~15 items), brute-force enumeration or branch-and-bound works fine. For larger auctions, you'll want an ILP solver.

**Bidding languages matter.** Asking agents to enumerate valuations for all `2^N` bundles is infeasible. Instead, support compact bidding languages:
- **XOR bids:** "I want A OR B, but not both." (At most one bid wins.)
- **OR bids:** "I'll take A, and separately I'll also take B." (Multiple bids can win.)
- **Additive bids with synergy bonuses:** "Each Iron is worth $10 to me, each Wood is worth $5, and if I get both Iron and Wood together, add a $100 crafting bonus."

For a game with crafting recipes, the last option maps naturally: agents express base commodity values plus recipe-completion bonuses.

**Timing and frequency.** Combinatorial auctions are inherently batch mechanisms — you collect bids, solve, and clear. They don't run continuously like a CDA. In your game, you might run a combinatorial auction once per "market day" or at the start of each crafting phase, while the CDA handles continuous commodity trading in between.

**Approximate solutions are fine.** The WDP is NP-hard in the worst case, but your game doesn't need perfect optimality. A greedy heuristic (sort bids by value density, accept greedily, skip conflicts) gets you 80% of the way there and runs in milliseconds. Use it for real-time play; save the ILP solver for end-of-round settlements where you have more compute budget.

---

## When to Use Combinatorial Auctions in Your Game

Use combinatorial auctions whenever agents have **complementary valuations** — when the combination of items is worth more than the parts:

- **Crafting systems:** Agents need specific bundles of materials to craft items. A combinatorial auction lets them bid for complete recipes.
- **Territory allocation:** Agents bidding on adjacent land tiles that are only valuable as a contiguous block.
- **Military logistics:** An army needs swords AND shields AND food to field a battalion. Winning swords without food is useless.
- **Trade route contracts:** A merchant values a set of trade routes that form a complete circuit, not isolated segments.

Don't use them for simple commodity trading (use a CDA or AMM), for single unique items (use an English or Vickrey auction), or for situations where items are independent and have no synergies (the computational overhead of combinatorial clearing buys you nothing).

The power of the combinatorial auction is that it lets your agents express what they *actually* want — complete bundles that serve a purpose — rather than forcing them to play a risky game of acquiring pieces one at a time and hoping the rest falls into place.
