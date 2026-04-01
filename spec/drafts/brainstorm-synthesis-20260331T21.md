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

-- Convenience: enumerate all commodities for fold-based computations
def allCommodities : List Commodity :=
  [.Manna, .Wood, .MithrilMetal, .Influence]
```

## 3. Value and Preferences (Utility & MRS)

To navigate the universe of goods, agents must be able to intrinsically value them.

*   **Utility ($U$)**: A measure mapping a bundle of goods to a real number ($\mathbb{R}$). It obeys three axioms:
    *   **Completeness**: For any two bundles $b_1, b_2$, either $U(b_1) \geq U(b_2)$ or $U(b_2) \geq U(b_1)$. Since $U$ maps into $\mathbb{R}$ and $\mathbb{R}$ is totally ordered, this is automatically satisfied by any real-valued function — it costs us nothing.
    *   **Transitivity**: If $U(b_1) \geq U(b_2)$ and $U(b_2) \geq U(b_3)$, then $U(b_1) \geq U(b_3)$. Again, this follows directly from transitivity of $\leq$ on $\mathbb{R}$. Any real-valued function is automatically transitive.
    *   **Non-Satiation (Strict Monotonicity)**: For any bundle $b$ and commodity $c$, adding a positive quantity of $c$ strictly increases utility: $U(b + \epsilon \cdot e_c) > U(b)$ for all $\epsilon > 0$. This is the substantive constraint — it encodes "more is always better." Unlike the first two, this does *not* come for free; it must be verified per utility function.

    **Enforcing Non-Satiation**: Since Completeness and Transitivity are automatic, the design question reduces to structural enforcement of Non-Satiation. For our Cobb-Douglas family (see below), this holds precisely when all exponents $\alpha_i > 0$ and all bundle quantities $x_i > 0$. We can enforce this at the type level by representing exponents as `{r : ℝ // r > 0}` and Bundle quantities as `{r : ℝ // r > 0}`.

*   **Marginal Utility ($MU$)**: The rate of change of utility as an agent consumes one more unit of a good (the partial derivative $\partial U / \partial x_i$).

*   **Marginal Rate of Substitution ($MRS_{i,j}$)**: The rate at which an agent can substitute good $x_j$ for good $x_i$ while maintaining the same level of utility. Formally, $MRS_{i,j} = MU_i / MU_j$, giving the amount of $x_j$ the agent would willingly give up per additional unit of $x_i$ gained.

### Canonical Utility Functions

Three canonical forms illuminate different behavioral assumptions:

**1. Cobb-Douglas** — $U(b) = \prod_{i} b_i^{\alpha_i}$

The workhorse of our simulation. Goods are *smoothly substitutable*: agents hold interior optima (they always want some of everything). The exponents $\alpha_i$ encode relative preference intensities. Non-satiation holds as long as $\alpha_i > 0$ and $b_i > 0$. MRS is always defined and equals $(\alpha_i / b_i) / (\alpha_j / b_j)$, which varies continuously with the bundle — giving rise to the familiar convex indifference curves.

```lean
-- Cobb-Douglas: U(b; α) = ∏ b(c)^α(c)
def cobbDouglasU (α : Commodity → ℝ) (b : Bundle) : Utility :=
  allCommodities.foldl (fun acc c => acc * (b c) ^ (α c)) 1

-- MU of good c under Cobb-Douglas: MU_c = α_c · U(b) / b(c)
-- Derived by differentiating the product rule: ∂/∂xc [∏ xᵢ^αᵢ] = αc · U / xc
def cobbDouglasMU (α : Commodity → ℝ) (c : Commodity) (b : Bundle) : ℝ :=
  α c * cobbDouglasU α b / b c

-- MRS_{ci, cj} under Cobb-Douglas = MU_ci / MU_cj = (α_ci · b_cj) / (α_cj · b_ci)
def cobbDouglasMRS (α : Commodity → ℝ) (ci cj : Commodity) (b : Bundle) : ℝ :=
  cobbDouglasMU α ci b / cobbDouglasMU α cj b
```

**2. Leontief (Perfect Complements)** — $U(b) = \min_i \bigl( b_i / r_i \bigr)$

Goods must be consumed in fixed proportions. A blacksmith needs exactly 1 Wood per 2 Manna — having 10 Wood and no Manna yields zero surplus Wood utility. MRS is undefined at the kink (the corner solution). This models production recipes or rigid consumption habits.

```lean
-- Leontief: U(b) = min over all c of (b(c) / ratio(c))
-- Initialise the fold at ⊤ (Float.inf) so every commodity is genuinely considered once.
def leontiefU (ratio : Commodity → ℝ) (b : Bundle) : Utility :=
  allCommodities.foldl (fun acc c => min acc (b c / ratio c)) Float.inf
```

**3. Quasilinear** — $U(b) = v(b_{c_0}) + b_{\text{numeraire}}$

Linear in the Numéraire, concave in a primary good $c_0$. Eliminates income effects on $c_0$: willingness to pay for $c_0$ is independent of wealth. This simplifies demand analysis because the marginal value of $c_0$ depends only on its own quantity, not on how rich the agent is.

```lean
-- Quasilinear: U(b) = v(b(primaryGood)) + b(MithrilMetal)
-- v is any concave function (e.g., √x or ln x)
def quasilinearU (v : ℝ → ℝ) (primaryGood : Commodity) (b : Bundle) : Utility :=
  v (b primaryGood) + b .MithrilMetal
```

### Indifference Curves and the $(N-1)$-Dimensional Preference Surface

An **indifference surface** at utility level $k$ is the set of all bundles yielding exactly $k$ utility:
$$\mathcal{I}(k) = \{ b \in \mathbb{R}^N_{>0} : U(b) = k \}$$

This surface is $(N-1)$-dimensional — one degree of freedom is consumed by holding utility fixed. For our 4-good economy, indifference "curves" are 3-dimensional surfaces embedded in $\mathbb{R}^4$. Rather than enumerating points on these surfaces, agents solve the **dual expenditure minimization problem**: given a target utility $k$ and prices $p$, find the cheapest bundle on $\mathcal{I}(k)$. This yields the Hicksian demand and is the computational backbone of `generate_optimal_trade` in Section 5.

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
  influenceLove : {r : ℝ // r > 0}  -- must be strictly positive to satisfy Non-Satiation
  mithrilLove   : {r : ℝ // r > 0}  -- must be strictly positive to satisfy Non-Satiation
  openness      : ℝ
  conscientiousness : ℝ

structure AgentState where
  hunger   : ℝ  -- in [0, 1]; 1 = starvation
  hp       : ℝ  -- in [0, 1]; 0 = dead
  exposure : ℝ  -- in [0, 1]; 1 = hypothermia
  fatigue  : ℝ  -- in [0, 1]; 1 = collapse

-- Derive Cobb-Douglas exponents from nature and physiological state.
-- Survival pressure (hunger, exposure) amplifies exponents for the relevant goods,
-- overriding long-term personality preferences when biological urgency is high.
def utilityExponents (nature : AgentNature) (state : AgentState) : Commodity → ℝ
  | .Manna        => 1 + state.hunger        -- hunger steepens food preference
  | .Wood         => 1 + state.exposure      -- exposure steepens shelter preference
  | .MithrilMetal => nature.mithrilLove.val
  | .Influence    => nature.influenceLove.val

-- Agent utility: Cobb-Douglas with state-modulated exponents
def agentU (b : Bundle) (nature : AgentNature) (state : AgentState) : Utility :=
  cobbDouglasU (utilityExponents nature state) b

-- Marginal utility of commodity c given current bundle, nature, and state
def agentMU (c : Commodity) (b : Bundle) (nature : AgentNature) (state : AgentState) : ℝ :=
  cobbDouglasMU (utilityExponents nature state) c b

-- MRS_{ci, cj}: units of cj the agent sacrifices per unit of ci at constant utility
def agentMRS (ci cj : Commodity) (b : Bundle) (nature : AgentNature) (state : AgentState) : ℝ :=
  cobbDouglasMRS (utilityExponents nature state) ci cj b

structure Agent where
  uuid        : String
  nature      : AgentNature
  state       : AgentState
  holdings    : Bundle  -- current inventory, updated each tick before trading
  endowment   : Bundle  -- per-tick in-kind income (Manna gathered, Wood chopped, MithrilMetal mined, etc.)
  trade_speed : ℝ       -- Action priority within the discrete tick queue

  U   : Bundle → AgentNature → AgentState → Utility := agentU
  MU  : Commodity → Bundle → AgentNature → AgentState → Utility := agentMU
  MRS : (ci cj : Commodity) → Bundle → AgentNature → AgentState → ℝ := agentMRS
```

If an agent's `state` deteriorates fully (e.g., invoking `dieOfStarvation` when `hunger` caps out), they are purged from the iteration loop.

## 5. Algorithmic Trading & The Simulation Loop

### How Market Prices Emerge from Agent MRS Values

Before examining the mechanics of trading, it is worth understanding where market prices come from in the first place. Prices are not exogenous — they are an emergent property of the heterogeneous MRS values held by agents across the economy.

The key insight is that each agent's MRS against the Numéraire is their personal **reservation price** for a good. An agent with $MRS_{\text{Wood}, \text{MithrilMetal}} = 8$ is willing to pay up to 8 MithrilMetal for 1 Wood. If the market price of Wood is 6, they will buy (gaining utility); if it is 10, they will sell.

The Continuous Double Auction (CDA) aggregates these reservation prices into buy and sell orderbooks. Buyers submit limit bids at or below their reservation price; sellers submit limit asks at or above theirs. A trade executes when a bid and ask cross. The **market-clearing price** settles at the point where the marginal buyer's reservation price meets the marginal seller's — in the language of general equilibrium, this is the **Walrasian equilibrium price**, at which every agent's MRS against the Numéraire equals the market price.

Prices therefore encode the MRS of the last (marginal) trader. When a sudden hunger spike raises every agent's $MRS_{\text{Manna}, \text{MithrilMetal}}$, bids for Manna surge and its market price rises to restore equilibrium. The CDA is simply the mechanism that aggregates and reveals this distributed information.

### The Discrete Time Architecture
1. **State Decay**: At the top of a tick, the environment cascades metabolic effects (hunger ticks up, cold drops HP if exposed). The agent's curve $U$ bends to urgently favor survival goods over long-term goals.
2. **Endowment Payout**: Each agent receives their per-tick in-kind income: `agent.holdings += agent.endowment`. Income is paid in real goods — Manna gathered from bushes, Wood chopped from trees, MithrilMetal mined — not purely in numeraire. This makes the sim an **endowment economy**: an agent's effective budget each tick is the market value of their updated holdings $p \cdot \text{holdings}$, not a fixed cash allowance. Crucially, this means wealth is price-dependent — a Manna gatherer becomes richer in real terms when the Manna price rises, because the value of their endowment appreciates.
3. **Execution Order**: Agents are sorted by their continuous `trade_speed` characteristic.
4. **Observation & Decision**: Each agent calculates their new structural $MRS$, observes the CDA orderbooks, and computes a target macro trade.

### Solving the Optimization Problem

On their turn, agents solve exactly one optimization problem: maximizing the utility of a prospective terminal bundle, subject to the constraint that the terminal bundle's market value cannot exceed the market value of their current holdings (which already include this tick's endowment payout). This is the standard endowment economy budget constraint — wealth is not a fixed cash figure but the price-dependent value of the goods the agent actually holds.

**Does the budget constraint bind?** The constraint is an inequality ($p \cdot \text{trade} \leq 0$), so agents are not formally required to spend their full endowment value. However, Non-Satiation (all Cobb-Douglas exponents $> 0$) guarantees that an agent always prefers more of any good — leaving budget slack is never optimal. As a result, the constraint holds at equality in practice: agents always exhaust the full market value of their holdings through reallocation. This is not an assumption but a theorem: it follows directly from the utility axioms.

The one exception is **speculative holding**: an agent might rationally keep excess holdings in a good if they expect its price to rise next tick. The base `generate_optimal_trade` optimizer is myopic (single-tick) and cannot reason about this. A speculative strategy would require a separate inter-temporal layer on top of the base optimizer.

```lean
-- Spot prices in terms of MithrilMetal, aggregated from the global CDA orderbooks.
-- Each price p(c) reflects the MRS of the marginal trader for commodity c.
def MarketPrices := Commodity → Quantity

-- Returns an ideal delta bundle (positive for buys, negative for sells).
-- PRECONDITION: agent.holdings must already include this tick's endowment payout
-- (step 2 of the tick loop) before this function is called.
--
-- Budget interpretation (endowment economy):
--   dot_product(trade, prices) ≤ 0
--   ⟺  p · (holdings + trade) ≤ p · holdings
--   ⟺  value of terminal bundle ≤ value of current holdings at market prices
--
-- Agents who are net sellers of a good (endowment > consumption) benefit when
-- its price rises; net buyers are hurt. This endowment income effect is the key
-- difference from a fixed-wage numeraire model and arises automatically from
-- the constraint once holdings are updated with in-kind income.
def generate_optimal_trade (agent : Agent) (prices : MarketPrices) : Bundle :=
  -- maximize: agent.U (agent.holdings + trade) agent.nature agent.state
  -- subject to: dot_product(trade, prices) ≤ 0          (budget balance: terminal value ≤ endowment value)
  --             ∀ c, agent.holdings c + trade c ≥ 0      (non-negativity: cannot sell what you don't own)
  let feasible := fun t =>
    dot_product t prices <= 0 ∧ ∀ c, agent.holdings c + t c ≥ 0
  argmax (fun t => agent.U (agent.holdings + t) agent.nature agent.state) feasible
```

### Trade Intention Slicing

Agents do not submit raw macro arrays to the system. The optimization subroutine outputs an ideal bundle delta (e.g., "I must acquire exactly 15 Wood and 3 Manna"). This intent is routed to the agent's internal **Trade Intention System**. This system algorithmically slices the macro-target into specific limit buy and sell layers directed straight into the global **Continuous Double Auction**. CDAs evaluate and match these granular orders purely if spreads cross.

*Note on AMM equivalence*: The optimization problem above closely mirrors how Automated Market Makers work. A constant-product AMM (e.g., Uniswap's $x \cdot y = k$) or a weighted AMM (e.g., Balancer's $\prod x_i^{w_i} = k$) is a market maker that enforces trades along an indifference surface. Notably, Balancer's formula with weights $w_i$ is mathematically isomorphic to a Cobb-Douglas utility function with exponents $\alpha_i = w_i$. This means agents in a bartering sub-economy could swap goods directly against an AMM pool whose price curve is derived from the Cobb-Douglas indifference geometry — an interesting future extension before a full CDA infrastructure is in place.

### Beyond the CDA: Combinatorial Auctions
*Future scope note:* While CDAs are pristine for singular commodities, Combinatorial Auctions permit bids for indivisible mixed lots of goods. Because evaluating these requires solving the NP-hard Winner Determination Problem (equivalent to set packing), they will operate completely outside the strict simulation tick-loop as asynchronous global events. Agents will compute their total max willingness to pay using their $U$-function, place closed bids, and the auction supervisor will execute distribution simultaneously while the global clock freezes.

## 6. Type-Level Guarantees & Verification

By modeling structural economic mechanics within dependent type theory (or robust Rust traits), we gain strong formal guarantees. In particular, encoding the feasibility constraint — `dot_product(trade, prices) ≤ 0` (budget balance) and `∀ c, holdings c + trade c ≥ 0` (non-negativity) — as proof obligations means that any proposed trade must carry evidence of both conditions. While this does not eliminate all runtime checks (numerical optimization still occurs at runtime), it structurally prevents ill-typed trades from being constructed in the first place.

Similarly, bounding state properties (e.g., `hunger ∈ [0, 1]`) at the type level isolates the complexity of multi-variate modeling, ensuring that shifting environments organically compel agents to mediate their idiosyncratic trait desires against rigid biological facts.

As noted in Section 3, Non-Satiation can be enforced structurally by requiring Cobb-Douglas exponents to inhabit the subtype `{r : ℝ // r > 0}` and bundle quantities to inhabit `{r : ℝ // r > 0}`. Under these constraints, a proof of Non-Satiation for `cobbDouglasU` is mechanically derivable — a well-typed agent *cannot* hold a utility function that violates the axioms.

## 7. Simulation as a Strategic Game

The simulation is not only a passive model — it is a platform for strategic play. Once the market loop is operational, a human (or AI controller) can assume the role of any agent and steer toward arbitrary strategic aims while the other agents pursue their own utility-maximizing ends:

*   **Survival run**: Maximize personal utility subject to keeping all physiological states below critical thresholds.
*   **Resource monopoly**: Accumulate enough Manna to drive up its price, then exploit the inelastic demand of starving agents.
*   **Political capture**: Corner the supply of Influence to dominate social hierarchies and extract rents.
*   **Market manipulation**: Engineering a liquidity crisis, triggering cascading sell-offs, or engineering a short squeeze on MithrilMetal.
*   **Cooperative play**: Coordinate with other agents to stabilize prices around Pareto-efficient outcomes, resisting defection.

This game-like framing also makes the simulator a natural testbed for studying real-world market failures: monopoly formation, speculative bubbles, hoarding under uncertainty, and the tragedy of the commons all emerge organically from the same agent-level utility mechanics.
