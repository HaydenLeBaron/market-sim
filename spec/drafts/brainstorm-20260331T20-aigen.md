# Goal Description

This implementation plan outlines a Literate Programming (Knuth-style) approach to building rudimentary algorithmic trading strategies for economic agents from first principles. By formalizing microeconomic concepts (utilities, Marginal Rate of Substitution) using functional, dependent type theory pseudocode (inspired strictly by Lean 4), we generate a rigorous foundation for agent behavior. 

Following the mathematical formalization, this plan specifies the precise Continuous Double Auction (CDA) market structure the simulation will rely on, and how agents execute complex Trade Intentions.

---

# Part 1: First-Principles Formalization (Literate Pseudocode)

We begin by defining the universe of goods. Based on your design docs, we only care about tradable **Commodities** (Products are excluded from trading in this MVP).

```lean
-- Provide basic types for quantities and utility
def Quantity := ℝ
def Utility  := ℝ

-- The finite universe of base commodities in our economy
inductive Commodity
  | Manna        -- Gathered from Bushes
  | Wood         -- Chopped from Trees
  | MithrilMetal -- Mined from MithrilOre (Numéraire)
  | Influence    -- Extracted from Reputation

-- A Bundle is an assignment of a Quantity to anything that can be held.
def Bundle := Commodity → Quantity

-- Addition of bundles (element-wise)
def Bundle.add (b1 b2 : Bundle) : Bundle :=
  fun c => b1 c + b2 c

-- Subtraction of bundles
def Bundle.sub (b1 b2 : Bundle) : Bundle :=
  fun c => b1 c - b2 c
```

### Agents and Preferences

An agent's behavior is driven by their static Nature and dynamic State. 
* Their static `AgentNature` establishes baseline psychological biases for esoteric assets.
* Their dynamic `AgentState` establishes survival pressures.
  * Desire for `Manna` is purely a function of `hp` and `hunger`.
  * Desire for `Wood` is driven directly by their `exposure` state.

```lean
structure AgentNature where
  influenceLove : ℝ
  mithrilLove   : ℝ

structure AgentState where
  hunger   : ℝ
  hp       : ℝ
  -- Exposure acts as the biological bias toward wanting Wood
  exposure : ℝ

structure Agent where
  uuid : String
  nature: AgentNature
  state : AgentState
  holdings : Bundle
  -- How fast the agent can act in the discrete tick priority queue
  trade_speed : ℝ
  
  -- The agent's utility function mapping a theoretical bundle AND their 
  -- physiological nature/state to a real number.
  U : Bundle → AgentNature → AgentState → Utility
  
  -- The Marginal Utility (MU) is the partial derivative of U with respect to a Commodity.
  MU : Commodity → Bundle → AgentNature → AgentState → Utility
  
  -- The Marginal Rate of Substitution (MRS) of Commodity A for Commodity B
  MRS : (a : Commodity) → (b : Commodity) → Bundle → AgentNature → AgentState → ℝ
```

### Algorithmic Trading: Macro Trade Intentions

Agents act on discrete time ticks. During their turn, they solve a structural optimization problem to find the single best `trade` bundle that maximizes their utility, subject to their budget.

```lean
-- Spot prices in terms of MithrilMetal
def MarketPrices := Commodity → Quantity 

-- Optimization Sub-Routine
-- Returns an ideal delta bundle (positive for buys, negative for sells)
def generate_optimal_trade (agent : Agent) (prices : MarketPrices) : Bundle := 
  -- Mathematically, they solve exactly one optimization problem per tick:
  -- maximize: agent.U (agent.holdings + trade) agent.nature agent.state
  -- subject to: dot_product(trade, prices) = 0
  
  -- Budget Constraint Explanation:
  -- dot_product(trade, prices) calculates the total net-cost of the intended trade.
  -- For a commodity the agent wants to BUY, trade[c] is positive.
  -- For a commodity the agent wants to SELL, trade[c] is negative. 
  -- Therefore, the sum of `(Quantity * Price)` across all commodities must be <= 0.
  -- This mathematically enforces that the agent only buys what they can fund via 
  -- selling their existing holdings. They cannot spend money they don't have.
  
  let budget_constraint := fun t => dot_product t prices <= 0
  argmax (fun t => agent.U (agent.holdings + t) agent.nature agent.state) budget_constraint
```

Once the `generate_optimal_trade` yields the ideal bundle delta, the agent **does not submit raw orders directly to the market**. 

Instead, they submit the idealized trade target to their **Trade Intention System**. This system represents their internal algorithmic execution suite. It decomposes the macro-trade into specific, complex trading strategies composed of individual atomic limit buy/sell orders that hit the Continuous Double Auction.

---

# Part 2: Architecture & Timing

1. **Discrete Ticks**: The primary simulation loop operates entirely on discrete ticks.
2. **State Updates**: At the start of a tick, the environment updates agent states (hunger increases, exposure drops hp if cold, etc.). This instantly cascades, shifting the agent's internal utility curve $U$ to skew heavily toward `Manna` or `Wood`.
3. **Turn Order**: Agents are sorted by `trade_speed` priority.
4. **Action Loop**: 
   - An agent calculates $MRS$ given their latest state and nature.
   - They look at the current CDA Orderbooks (the spread).
   - They run `generate_optimal_trade` to find their maximal utility delta.
   - They generate a **Trade Intention** seeking to fulfill this delta.
   - The Trade Intention Strategy executes, slicing limit/market orders into the CDA.
   - The CDA processes executions sequentially if spreads cross.

---

# Part 3: Future Suggestive Notes

### Combinatorial Auctions
*This feature is out of scope for the current MVP, but mapping its math ensures our dependent-typing supports it natively later.*
While CDAs are great for single commodities, combinatorial auctions allow agents to bid on indivisible lots of goods. 
Because these rely on NP-Complete Winner Determination solvers, they cannot exist in the primary tick-loop. They will be pushed to a completely separate "game" / interface.
They will operate as scheduled, rare events. Agents will compute their expected utility delta for the *entire indivisible lot*, derive a max willingness to pay in `MithrilMetal`, and submit closed bids. The auctioneer freezes, settles the knapsack problem, and distributes the lots in exactly one tick.

---

## Verification Plan

### Lean 4 Structural Verification
Since there are no automated tests, validation will occur purely by manually writing and reading Lean 4 typed structural math. By maintaining strict dependent-type boundaries, we can visually guarantee that an agent cannot submit an action that violates their budget constraint, and that their objective function dynamically forces them to buy `Manna` and `Wood` depending on their `AgentState` needs.
