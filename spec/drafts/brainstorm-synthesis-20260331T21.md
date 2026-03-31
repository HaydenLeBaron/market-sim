# Brainstorm Synthesis: Economic Agent Model & Market Architecture

*Synthesized from `brainstorm-20260331T12.md` and `brainstorm-20260331T20-aigen.md`.*

## 1. Introduction

This document outlines a formal, first-principles approach to building a simulation of economic agents. By combining deep microeconomic theory (Utilities, Marginal Rate of Substitution) with strong architectural patterns (inspired by Rust and Lean 4-style dependent type theory), we define a rigorous foundation for agent behavior, market dynamics, and algorithmic execution strategies.

## 2. The Universe of Goods

**Intuitively**: *Goods* (or *Commodities*) are the set of resources that economic agents consume or trade to maximize their survival and happiness.

**Formally**: The economy consists of a finite set of base commodities. A `Bundle` is defined as an assignment of a `Quantity` to these commodities.

```lean
-- Provide basic types for quantities and utility
def Quantity := ℝ
def Utility  := ℝ

-- The finite universe of base commodities in our economy
inductive Commodity
  | Manna        -- Gathered from Bushes (Food)
  | Wood         -- Chopped from Trees (Shelter/Fuel)
  | MithrilMetal -- Mined from MithrilOre (Numéraire)
  | Influence    -- Extracted from Reputation (Status)

-- A Bundle is an assignment of a Quantity to anything that can be held.
def Bundle := Commodity → Quantity

-- Addition and Subtraction of bundles happens element-wise
def Bundle.add (b1 b2 : Bundle) : Bundle := fun c => b1 c + b2 c
def Bundle.sub (b1 b2 : Bundle) : Bundle := fun c => b1 c - b2 c
```

## 3. Value and Preferences (Utility & MRS)

To navigate the universe of goods, agents must be able to intrinsically value them.

*   **Utility ($U$)**: A measure mapping a bundle of goods to a real number ($\mathbb{R}$). It obeys axioms of Completeness, Transitivity, and Non-satiation.
*   **Marginal Utility ($MU$)**: The rate of change of utility as an agent consumes one more unit of a good (the partial derivative $\partial U / \partial x_i$).
*   **Marginal Rate of Substitution ($MRS_{i,j}$)**: The rate at which an agent can substitute good $x_j$ for good $x_i$ while maintaining the same level of utility. Formally, $MRS_{i,j} = MU_i / MU_j$, giving the amount of $x_j$ the agent would willingly give up per additional unit of $x_i$ gained.

### The Computational Efficiency of a Numéraire

In a pure bartering economy, tracking pairwise MRS combinations across the network results in a combinatorial explosion ($\binom{n}{2} = n(n-1)/2$ markets, i.e. $O(n^2)$). By choosing a **Numéraire** (e.g., `MithrilMetal` or `Gold`), we collapse these combinations into a simple $(N-1)$-dimensional spot price vector (the Numéraire trivially prices itself at 1). The price of any good mathematically maps to its MRS against the Numéraire.

Qualities of a sound Numéraire include fungibility, scarcity, stability, and portability. While the MVP specifically hardcodes `MithrilMetal` as the Numéraire, future design phases can explore emergent monetary structures where agents naturally settle on a unit of account from an initial bartering state.

## 4. Agent Architecture

An agent's behavior is modeled by merging two core subsystems: their static (immutable) **Nature** and their dynamic (mutable) **State**. Together, these physiological and psychological inputs shape the agent's utility function, heavily swaying their market preferences.

*   **Agent Nature**: Baseline, immutable personality traits (e.g., Big-5 traits like Openness, Conscientiousness) and innate biases (e.g., `influenceLove` for Status, or inherent `securityBias`).
*   **Agent State**: Dynamic, rapidly decaying physiological variables (e.g., `hp`, `hunger`, `exposure`, `fatigue`, `aggrievedness`) representing survival pressures.

These states output composite biases. For instance, escalating `hunger` steepens the Marginal Utility for `Manna`; surging `exposure` does the same for `Wood`.

```lean
structure AgentNature where
  influenceLove : ℝ
  mithrilLove   : ℝ
  openness      : ℝ
  conscientiousness : ℝ

structure AgentState where
  hunger   : ℝ
  hp       : ℝ
  exposure : ℝ
  fatigue  : ℝ

structure Agent where
  uuid : String
  nature: AgentNature
  state : AgentState
  holdings : Bundle
  trade_speed : ℝ -- Action priority within the discrete tick queue
  
  -- Utility dynamically shifts based on shifting physiological states
  U : Bundle → AgentNature → AgentState → Utility
  MU : Commodity → Bundle → AgentNature → AgentState → Utility
  MRS : (a b : Commodity) → Bundle → AgentNature → AgentState → ℝ
```

If an agent's `state` deteriorates fully (e.g., invoking `dieOfStarvation` when `hunger` caps out), they are purged from the iteration loop.

## 5. Algorithmic Trading & The Simulation Loop

### The Discrete Time Architecture
1. **State Decay**: At the top of a tick, the environment cascades metabolic effects (hunger ticks up, cold drops HP if exposed). The agent's curve $U$ bends to urgently favor survival goods over long-term goals.
2. **Execution Order**: Agents are sorted by their continuous `trade_speed` characteristic.
3. **Observation & Decision**: Each agent calculates their new structural $MRS$, observes the Continuous Double Auction (CDA) orderbooks, and computes a target macro trade.

### Solving the Optimization Problem

On their turn, agents solve exactly one optimization problem: maximizing the utility of a prospective terminal bundle, tightly constrained by their strict budgetary limits.

```lean
-- Spot prices read from the global CDA in terms of MithrilMetal
def MarketPrices := Commodity → Quantity 

-- Returns an ideal delta bundle (positive for buys, negative for sells)
def generate_optimal_trade (agent : Agent) (prices : MarketPrices) : Bundle := 
  -- maximize: agent.U (agent.holdings + trade) agent.nature agent.state
  -- subject to: dot_product(trade, prices) ≤ 0  (net expenditure cannot exceed proceeds)
  let budget_constraint := fun t => dot_product t prices <= 0
  argmax (fun t => agent.U (agent.holdings + t) agent.nature agent.state) budget_constraint
```

### Trade Intention Slicing

Agents do not submit raw macro arrays to the system. The optimization subroutine outputs an ideal bundle delta (e.g., "I must acquire exactly 15 Wood and 3 Manna"). This intent is routed to the agent's internal **Trade Intention System**. This system algorithmically slices the macro-target into specific limit buy and sell layers directed straight into the global **Continuous Double Auction**. CDAs evaluate and match these granular orders purely if spreads cross.

### Beyond the CDA: Combinatorial Auctions
*Future scope note:* While CDAs are pristine for singular commodities, Combinatorial Auctions permit bids for indivisible mixed lots of goods. Because evaluating these requires solving the NP-hard Winner Determination Problem (equivalent to set packing), they will operate completely outside the strict simulation tick-loop as asynchronous global events. Agents will compute their total max willingness to pay using their $U$-function, place closed bids, and the auction supervisor will execute distribution simultaneously while the global clock freezes.

## 6. Type-Level Guarantees & Verification

By modeling structural economic mechanics within dependent type theory (or robust Rust traits), we gain strong formal guarantees. In particular, encoding the budget constraint `dot_product(trade, prices) ≤ 0` as a proof obligation means that any proposed trade must carry evidence of feasibility. While this does not eliminate all runtime checks (numerical optimization still occurs at runtime), it structurally prevents ill-typed trades from being constructed in the first place.

Similarly, bounding state properties (e.g., `hunger ∈ [0, 1]`) at the type level isolates the complexity of multi-variate modeling, ensuring that shifting environments organically compel agents to mediate their idiosyncratic trait desires against rigid biological facts.
