# The Continuous Double Auction: Building a Real-Time Marketplace for Your Game Agents

You have a hundred AI agents. Each one holds an inventory — wood, iron, food, tools — and each one wants something it doesn't have. You need a mechanism that lets any agent, at any moment, post an offer to buy or sell, and have that offer matched instantly if someone on the other side is willing.

That mechanism is the **Continuous Double Auction (CDA)**, and it is the single most important market structure you will implement.

Every major stock exchange, every forex platform, every crypto order book — they all run on a CDA. It is the engine that turns chaotic, asynchronous, self-interested behavior into a functioning price system. If your simulator has only one market type, make it this one.

---

## What Is It?

A CDA is a market that is always open. Buyers submit **bids** (offers to buy at a given price), sellers submit **asks** (offers to sell at a given price), and whenever a bid meets or exceeds an ask, a trade executes immediately.

There is no auctioneer calling out prices. There is no round that ends. The market simply *runs*, processing orders as they arrive, and a price emerges from the aggregate behavior of all participants.

---

## The Order Book

The core data structure is the **order book**. It has two sides:

**Bid side (buy orders):** Sorted by price, highest first. At equal prices, the order that arrived earlier has priority. This is called **price-time priority**.

**Ask side (sell orders):** Sorted by price, lowest first. Same time-priority rule.

The gap between the best (highest) bid and the best (lowest) ask is the **spread**. A narrow spread means the market is liquid — buyers and sellers nearly agree on price. A wide spread means they don't, and trading will be sluggish.

```
Bid Side               Ask Side
-----------            -----------
Blacksmith: $52 x 3    Lumberjack: $54 x 10
Farmer:     $50 x 5    Miner:      $56 x 8
Scout:      $48 x 2    Merchant:   $60 x 4

        Spread = $54 - $52 = $2
```

---

## Matching Rules

When a new order arrives, the engine checks whether it can execute against the opposite side of the book:

1. **Incoming bid >= best ask?** Trade executes at the ask price (the resting order's price).
2. **Incoming ask <= best bid?** Trade executes at the bid price.
3. **Neither?** The order rests on the book as a **limit order**, waiting for a future counterparty.

A **market order** — an order with no price limit — simply grabs the best available resting order and trades immediately.

The critical detail: **the resting order's price wins**. If the Lumberjack has an ask sitting at $54 and a new bid comes in at $57, the trade happens at $54, not $57. The incoming order crosses the spread and takes what's available.

---

## Walkthrough: Agents Trading Wood Planks

Let's run through a concrete sequence. The order book starts empty.

**Step 1:** The Blacksmith needs wood and places a **bid of $49 for 10 planks**.

| Bid Side | Ask Side |
|---|---|
| Blacksmith: $49 x 10 | *(empty)* |

No trade. There's nobody selling.

**Step 2:** The Lumberjack has surplus wood and places an **ask of $51 for 10 planks**.

| Bid Side | Ask Side |
|---|---|
| Blacksmith: $49 x 10 | Lumberjack: $51 x 10 |

No trade. The bid ($49) is less than the ask ($51). Spread = $2.

**Step 3:** A Merchant arrives and places a **bid of $51 for 5 planks**.

The Merchant's bid ($51) >= the Lumberjack's ask ($51). **Trade executes: 5 planks at $51.**

| Bid Side | Ask Side |
|---|---|
| Blacksmith: $49 x 10 | Lumberjack: $51 x 5 *(remaining)* |

**Step 4:** A Farmer places an **ask of $48 for 8 planks**.

The Farmer's ask ($48) <= the Blacksmith's bid ($49). **Trade executes: 8 planks at $49.**

| Bid Side | Ask Side |
|---|---|
| Blacksmith: $49 x 2 *(remaining)* | Lumberjack: $51 x 5 |

**Step 5:** A Miner places an **ask of $50 for 6 planks**.

The Miner's ask ($50) > the Blacksmith's bid ($49). No trade. The order rests.

| Bid Side | Ask Side |
|---|---|
| Blacksmith: $49 x 2 | Miner: $50 x 6 |
| | Lumberjack: $51 x 5 |

The spread has tightened to $1. The market is getting more liquid.

**Trade log:**

| Buyer | Seller | Price | Quantity |
|---|---|---|---|
| Merchant | Lumberjack | $51 | 5 |
| Blacksmith | Farmer | $49 | 8 |

---

## Why the CDA Works for Your Simulator

**It converges to equilibrium.** This is the remarkable result demonstrated by Vernon Smith (Nobel Prize, 2002): even with a small number of poorly-informed agents, a CDA reliably converges to the competitive equilibrium price — the price where supply meets demand. Your agents don't need to be brilliant. They just need to place orders based on their own local knowledge of what they need and what they're willing to pay.

**It provides continuous price discovery.** Unlike a batch auction that clears once per round, a CDA updates prices in real time. When a drought hits your game world and food becomes scarce, the CDA price of food will spike immediately as agents scramble to buy and sellers hold out for higher prices. This gives every agent in the system an up-to-the-tick signal about relative scarcity.

**It handles heterogeneous urgency.** Some agents desperately need iron *right now* because they're under attack and need to forge swords. They'll submit market orders and pay the spread. Other agents are patient — they'll post limit orders and wait. The CDA lets both coexist, with the spread acting as the price of immediacy.

---

## Implementation Considerations

**Data structure choice matters.** The order book needs fast insertion, deletion, and lookup of the best bid/ask. A common approach is a sorted map (e.g., `BTreeMap` in Rust) keyed by price level, with each level holding a queue of orders at that price (enforcing time priority within the level).

**Partial fills are the norm.** An incoming order for 20 units might match against three resting orders of 5, 8, and 7 units. Your matching engine must walk the book, filling against each resting order in priority order until the incoming order is fully satisfied or the book is exhausted.

**Ticks vs. continuous time.** In a real exchange, orders arrive in nanoseconds. In a game simulation, you likely run discrete ticks. During each tick, multiple agents may submit orders. You have a design choice: process them in random order (simulating asynchronous arrival), in a fixed order (simulating priority), or collect them all and run a mini batch auction per tick. The first option most closely approximates a true CDA.

**The spread is your health metric.** A healthy market has a tight spread and deep order books on both sides. If your CDA for iron has a spread of $0.50 on a $50 item, agents are finding counterparties easily. If the spread is $25, the market is broken — probably too few agents trading iron, or agents' valuations are wildly misaligned. Monitor spreads to diagnose your economy.

---

## When to Use a CDA in Your Game

Use a CDA as the **default market** for fungible commodities — wood, stone, food, iron, gold. Any good where one unit is interchangeable with another is a natural fit.

The CDA is *not* ideal for unique items (use an English or sealed-bid auction), bundles of items with synergies (use a combinatorial auction), or situations where agents should be forced to commit resources just to compete (use an all-pay auction). We'll cover those in later posts.

But for the bread and butter of your simulated economy — the constant, fluid exchange of raw materials and basic goods — the CDA is the mechanism that makes it all work.
