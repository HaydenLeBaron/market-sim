# Market Simulator: Project Vision Proposal

**Date:** 2026-03-30
**Status:** Draft Proposal

---

## Executive Summary

`market-sim` is a Rust crate that implements a suite of classical market and auction mechanisms -- from simple bartering to continuous double auctions -- behind a common, composable interface. It is designed as a **library first**: consumers import it as a dependency and wire it into their own simulations, games, or research tools. The primary motivating use case is integration with an existing game project whose AI agents hold inventories of typed items and need to trade autonomously with each other and with human players.

Beyond the core library, `market-sim` includes a companion **web-based visualization** layer. A server exposes live market state, and a browser client renders real-time order books, trade feeds, agent portfolios, and price charts. Crucially, the web client is not just a passive viewer -- a human user can place bids and asks directly, participating in the simulated markets alongside AI agents.

Together, the library crate and the visualization layer make `market-sim` useful for three audiences: game developers who need an in-game economy engine, researchers who want to experiment with market mechanism design, and educators who want interactive demonstrations of auction theory.

---

## Motivation & Problem Statement

### The game integration problem

The author maintains a separate game project in which autonomous agents hold inventories of heterogeneous items (wood, stone, food, tools, etc.). These agents need to trade with one another to acquire resources they lack. Hard-coding a single exchange mechanism (e.g., fixed prices set by the developer) is brittle and uninteresting. What is needed is a pluggable market layer that can be swapped, configured, and observed -- without coupling the game's core logic to any single trading mechanism.

`market-sim` solves this by providing a library of market types behind a common interface. The game project depends on the crate, instantiates one or more markets, registers its agents, and lets them trade. Switching from a continuous double auction to a sealed-bid auction is a configuration change, not a rewrite.

### The experimentation and education problem

Different market mechanisms have different properties: some are incentive-compatible, some reveal more information, some converge faster to equilibrium prices. Understanding these tradeoffs in the abstract is useful; *seeing them in action* with live agents placing real orders is far more illuminating. The visualization component turns `market-sim` into a hands-on learning tool and experimental sandbox.

---

## Scope: Supported Market Types

`market-sim` targets the following mechanisms (see `domain-knowledge.md` for detailed explanations and examples of each):

1. **Traditional Bartering** -- Direct exchange of goods between two parties without money.
2. **English (Ascending) Auction** -- Open bidding with ascending prices; highest bidder wins.
3. **Dutch (Descending) Auction** -- Price falls from a high starting point; first claimant wins.
4. **First-Price Sealed-Bid Auction** -- Secret bids; highest bidder wins and pays their bid.
5. **Second-Price (Vickrey) Sealed-Bid Auction** -- Secret bids; highest bidder wins but pays the second-highest bid.
6. **Call (Batch) Double Auction** -- Bids and asks are collected over a window, then cleared at a single price.
7. **Continuous Double Auction (CDA)** -- An ongoing order book where trades execute whenever a bid meets an ask.

The library is designed to be **extensible**. Users can implement additional market types (combinatorial auctions, prediction markets, etc.) against the same common interface and plug them into the system.

---

## Core Concept: The Library Crate

### Usage as a Dependency

`market-sim` is a Rust crate, importable via `Cargo.toml`. It has no mandatory runtime, GUI framework, or network dependency -- the core library is pure logic. The visualization server is an optional feature or a companion binary crate.

The primary consumers are other Rust projects, especially games and agent-based simulations. The target integration looks like:

```toml
[dependencies]
market-sim = "0.1"
```

The consuming project creates markets, registers agents, submits orders, and observes outcomes -- all through the library's public API.

### Key Concepts

These are the core nouns of the system, described at the product level:

- **Agent** -- An entity that participates in markets. It has an identity, an inventory, and a trading strategy. Agents can be AI-controlled (autonomous) or human-controlled (via the web client).
- **Inventory** -- A collection of typed, quantified holdings belonging to an agent (e.g., 50 wood, 12 iron, 3 potions). The inventory model is generic so that consumers define their own item types.
- **Market** -- A venue where agents interact according to a specific mechanism's rules. A simulation can run multiple markets concurrently (e.g., a CDA for common commodities and a sealed-bid auction for rare items).
- **Order** -- The primitive that an agent submits to a market. Depending on the mechanism, this might be a bid, an ask, a sealed bid, or a barter proposal. Each market type defines its own order semantics.
- **Trade** -- A completed transaction recording what was exchanged, between whom, and at what price (or exchange ratio). Trades update the involved agents' inventories.

### Pluggable Mechanisms

Each market type implements a common interface. The library consumer chooses which market type(s) to instantiate and can run multiple markets simultaneously within the same simulation. For example, a game world might operate:

- A **CDA** for high-volume commodity trading (wood, stone, food)
- A **Vickrey auction** for rare artifact sales
- **Bartering** for informal player-to-player exchanges

All of these can coexist, and agents can participate across multiple markets.

---

## Feature: Trade Intention Rebalancer

Many autonomous agents do not think in terms of individual buy/sell orders. Instead, they have a *goal state* for their inventory -- a target allocation -- and they want the system to figure out what trades are needed to get there. The Trade Intention Rebalancer bridges this gap.

### How it works

1. **Target specification:** An agent expresses a desired portfolio as ratios or absolute quantities. For example: *"I want my holdings to be roughly 40% wood, 30% stone, 30% food."*

2. **Delta computation:** The system compares the agent's current inventory (the "is") to the target (the "ought") and computes the difference -- what needs to be acquired and what can be sold.

3. **Intention generation:** The delta is translated into a set of **trade intentions**: abstract directives like "acquire ~20 wood" and "sell ~10 stone."

4. **Order translation:** Trade intentions are converted into concrete order primitives appropriate for the market the agent is participating in:
   - In a CDA: limit orders at reasonable prices based on recent market data.
   - In a sealed-bid auction: a bid calibrated to the agent's valuation.
   - In a barter scenario: a proposal offering surplus items for needed ones.

This is a **high-level convenience layer** on top of the core market primitives. It is especially useful for autonomous agents in game worlds that think in terms of survival goals (keep food above a threshold, accumulate building materials) rather than individual trade executions.

---

## Visualization: Web Client

### Overview

`market-sim` includes a web-based visualization tool for observing and interacting with simulated markets in real time. The server component exposes market state over WebSocket (or HTTP for snapshots), and a browser-based client renders it.

### What the user sees

- **Live order books** for each active market -- bids and asks, depth, and spread.
- **Trade history / ticker feed** -- a scrolling log of executed trades with price, quantity, and participants.
- **Agent dashboards** -- inventory contents, portfolio allocation, recent trading activity, and (where applicable) the agent's current trade intentions.
- **Price charts** -- price over time, volume bars, bid-ask spread history.
- **Market selector** -- switch between different active markets (e.g., the wood CDA, the artifact auction).

### Human participation

The web client is not merely a viewer. A human user can:

- **Place bids and asks** in any active market, just like an AI agent.
- **Participate in auctions** -- submit sealed bids, join English auctions, claim items in Dutch auctions.
- **Propose barter trades** with specific agents.
- **Set trade intentions** using the rebalancer -- define a target portfolio and let the system generate orders on their behalf.

This makes `market-sim` useful as a testbed where a human can trade alongside AI agents, test strategies, or simply experience how different market mechanisms feel from a participant's perspective.

### Not in scope for v1

- Native desktop or mobile clients -- the browser is the only supported client.
- Sub-millisecond rendering or trading latency -- this is a simulator, not a production exchange.
- Multi-user collaboration (multiple humans in the same simulation) -- a potential future enhancement.

---

## Non-Goals & Boundaries

- **Not a production trading platform.** There is no real money, no regulatory compliance, no security hardening for adversarial participants.
- **Not an architecture document.** This proposal deliberately avoids specifying module layout, data structures, Rust trait hierarchies, or internal design decisions. Those belong in a separate technical design document.
- **Performance target:** "Comfortable simulation with hundreds of agents and several concurrent markets." Not exchange-grade nanosecond latency or millions of orders per second.
- **Not a standalone application.** The core value is the library crate. The visualization server and web client are companion tools, not the primary product.

---

## Potential Future Work

- **Zero-knowledge proof auctions:** An exploratory direction -- using ZK-proofs (e.g., via [Midnight.dev](https://midnight.dev) or similar tooling) to implement sealed auctions where neither the winning bid nor the winner's identity is revealed to other participants. This would be most naturally applicable to sealed-bid auction variants. This is a low-priority idea and is not part of the core vision; it may or may not be pursued.
- **Additional market types:** Combinatorial auctions, prediction markets, all-pay auctions, and other exotic mechanisms.
- **Distributed simulation:** Running agents on different machines, communicating over a network -- useful for large-scale experiments or multiplayer game integration.
- **Game engine integration guides:** Documentation and example code for wiring `market-sim` into popular game engines or ECS frameworks.

---

## Summary

`market-sim` rests on three pillars:

1. **A Rust library crate** with pluggable, composable market mechanisms -- importable as a dependency by games, simulations, and research tools.
2. **A trade intention rebalancer** that lets autonomous agents think in terms of portfolio goals and have those goals automatically translated into market-appropriate orders.
3. **A web-based visualization client** where users can observe live markets and participate as traders alongside AI agents.

The immediate driving use case is integration into a game with inventory-bearing agents, but the design is general enough to serve anyone interested in simulating, studying, or teaching market dynamics.
