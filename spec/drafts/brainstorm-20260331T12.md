# Brainstorm: 2026-03-31T12:35

## Goods

**Intuitively**: *Goods* are the set of things that economic agents want to consume. 

**Formally**: 
In later analyses, N goods form an N dimensional space in which to calculate N-1 dimensional indifference curves.
```rs /math pseudocode
// Goods are a Sum type
type Good = | x1 | x2 | x3

// And the set of all goods is each variant in the sum type
let goods::Set<Good> := {x1, x2, x3, ..., xn}
```

## Utility (across N goods)

**Intuitively**: *Utility* is a measure of the total satisfaction a consumer gets from consuming a bundle of goods. In reality, Utility is an Ordinal, not a Cardinal concept. Yet at times it is useful and simpler to treat it as a Cardinal concept.

**Formally**:
```rs /math pseudocode
/** * *Utility* is a function that maps a bundle of goods to a real number, such that if bundle A is preferred to bundle B, then the utility of bundle A is greater than the utility of bundle B.*/
fn U(goods.{x1, x2, x3, ..., xn}) := (...):: ℝ
```

and again, more concisely:

```rs / math pseudocode
fn U(goods) → ℝ := ...
 ```

An economic agent can have an *arbitrary* utility function (one of its properties) that satisfies the following,
**Axioms of Utility**:
1. Completeness: For any two bundles A and B, the agent can compare them and determine whether they prefer A to B, B to A, or are indifferent between them.
2. Transitivity: For any three bundles A, B, and C, if the agent prefers A to B and B to C, then the agent prefers A to C.
3. Non-satiation: For any bundle A, there exists a bundle B that the agent prefers to A.

## Marginal Utility 

**Intuitively**: *Marginal Utility* is the rate of change of utility as you consume one more unit of a good.

**Formally**:
```rs /math pseudocode
/** The marginal utility of good xi across n goods is the partial derivative of the utility function with respect to good xi */
fn MU(of_xi ∊ goods, goods.{x1, x2, x3, ..., xn}) := 
  ∂U(x1, x2, x3, ..., xn) / ∂xi
  where i ∈ {1, 2, 3, ..., n}
```

And again, but more concisely:
```rs / math pseudocode
fn MU(of∊goods, goods) := ∂U(goods)/∂x
```

## Marginal Rate of Substitution

### Pairise MRS

**Intuitively**: *The marginal rate of substitution* of good xi for good xj is the rate at which a consumer is willing to trade good xi for good xj while maintaining the same level of utility.

**Formally**:
```rs /math pseudocode
/** The marginal rate of substitution of good xi space for good xj (in an n-dimensional space) is the ratio of the marginal utilities of good xi and good xj */
fn PairwiseMRS_xi_xj(x1, x2, x3, ..., xn) := 
  MU_xi(x1, x2, x3, ..., xn) / MU_xj(x1, x2, x3, ..., xn)
  where i, j ∈ {1, 2, 3, ..., n}
```

And again, but more concisely:
```rs / math pseudocode
fn PairwiseMRS(of_xi∊goods, for_xj∊goods, goods) :=
  MU(of_xi, goods) / MU(for_xj, goods)
```


### MRS Combinations in N-Dimensions

MRS is essentially a pairwise concept answering the question "Between two goods, at what rate would I be willing to trade two goods with no change in happiness (utility)". It therefore encodes indifference between two resources.

But we may also ask the question, "What is the MRS for each pairwise combination of goods across N goods?"

`∃n Goods`
=> `∃ Choose(n, 2)=n(n-1)/2` pairwise MRS combinations.

In a bartering economy, there would be a combinatorial explosion of markets: there would be `Choose(n, 2)` bartering pairs in total.
| Goods | Markets |
| ----- | ------- |
| 10    | 45      |
| 20    | 190     |
| 30    | 435     |
| 100   | 4,950   |


One of the beautiful things about a monetary economy is its computational efficiency. Choosing a numeraire (unit of account) good (gold, seashells, or fiat currency--for example) collapses the combinatorial explosion of pairwise MRS combinations into a simple N-dimensional price system. 

Rather than compute a dictionary of all `Choose(n, 2)` trading pairs, we can simply calculate an `n-1` dimensional vector/dictionary against a numeraire good (which is, itself, the `n`th good--which would always be trivially valued 1:1 against itself, like any good).


```rs math/pseudocode
/** What is the price of each good∊Goods in terms of the Good x3?
That would be the MRS of each good against x3.

prices is a function that computes a vector 
(dictionary with named fields, for readability) of the Pairwise MRS of each good against a numeraire good. */
fn prices(goods::Set<Good>, numeraire=x3::Good) :=
{
  x1 := PairwiseMRS(of_xi=x1, for_xj=x3, goods),
  x2 := PairwiseMRS(of_xi=x2, for_xj=x3, goods),
  // NOTE: x3 is the numeraire so it will be trivially equal to 1
  x3 := PairwiseMRS(of_xi=x3, for_xj=x3, goods) 
  ....
}
```

Similarly, if we don't have a numeraire we can compute the total
bartering vector for each pair of goods.

```rs math/pseudocode
fn barterPairs(goods:Set<Good>) :-
{
  x1x2 := PairwiseMRS(of_xi=x1, for_xj=x2, goods),
  x1x3 := PairwiseMRS(of_xi=x1, for_xj=x3, goods),
  x2x3 := PairwiseMRS(of_xi=x2, for_xj=x3, goods),
  ...
  // Of course, we will experience combinatorial explosion with many goods
}
```

But there is a much simpler way to calculate barter pairs in a monetary economy as the price vector implicitly encodes all bartering pairs. Even though you only explicitly compute the N pairs against money, you ahven't lost the other combinations!

```rs math/pseudocode
fn MRS(of_xi∊goods, for_xj∊goods, prices::Map<Good, ℝ>) 
  prices[xi] / prices[xj]
```



### Qualities of good Money

What are the requirements to be a good numeraire?
- fungibility (how can you be a unit of account without it)?
- scarcity (to resist inflation)
- stability (against a basket of other goods required for living, to ensure predictability of the future)
- portability (to facilitate trade)

Gold has more or less all of these qualities.

But in our simulation, different societies (united by a single market) could have different numeraires--or no numeraire at all if they are a bartering instead of a monetary economy.


## Connecting it all: Example

Suppose we have a world that contains `Goods` and `Agents`.


```rs pseudocode
type Good =
| Gold // Will be Numeraire in our Economy as a simplifying assumption. Later we can parameterize our economy and decide whether it will be a monetary economy or bartering economy, so we can determine how what Agent Nature is required for a monetary economy to arise as an emergent phenomenon.
| Food // Will be the only thing that can be consumed
| Shelter // Will be the only thing that can be used to protect from the elements. This is a desire to rent or buy property
| Water // Will be the only thing that can be used to quench thirst
| Status // Represented and Traded as securities within an society/market/economy/tribe/polity For not economies/markets/clans/polity will be tightly coupled as a simplifying assumption


fn goods 't -> Set<'t> := ... // generates the set of all `Good`s from a sum type representing goods

// or we can simply generate it by hand

```

```rs pseudcocode

/**
Using Big-5 for now because I know there is a lot of empirical research on how various personality traits influence behavior.
*/
type Personality = 
{
  openness ∊ 1-100%,
  conscientiousness ∊ 1-100%,
  extraversion ∊ 1-100%,
  agreeableness ∊ 1-100%,
  neuroticism ∊ 1-100%,
}

/**
The hardest thing when modeling an agent is deciding
- what should be mutable/immutable
- which attributes depend on which (independence vs dependence)
- keeping the model simple enough to understand and debug when things go wrong
*/
type agent = 
{
  uuid: string,

  /** Nature represents the agent's independent, innate characteristics, independent of their environment or experiences. Changing nature is either impossible or rare.  */
  nature:
  {
    personality: Personality,
    securityBias ∊ 0-100%
  }

  /** Independent, stateful characteristics of an agent that change significantly over time. */
  stateful:
  {
    mut isAlive: Bool,
    mut isAsleep: Bool,
    mut currHunger ∊  0-100%, // accumulating bias for food
    mut currThirst ∊ 0-100%,  // accumulating bias for water
    // Abstract representation of how much they have experienced wrongdoing at the hands of others, independent of their personality or how they feel about it.
    mut currExposure ∊ 0-100%, // accumulating bias for shelter
    mut currFatigue ∊ 0-100%, // accumulating bias for rest/sleep
    mut currAggrievedness ∊  0-100%, 
  }
  /*
  */
  composite:
  {
    // Bias towards holding numeraire Good of their market/economy
    fn numeraireBias(
      self.nature.personality.conscientiousness,
      self.nature.securityBias
      ) → ℝ;

    // Bias towards wanting to kill someone
    fn bloodlust(
      self.nature.personality.agreeableness, 
      self.nature.personality.neuroticism, 
      self.stateful.currAggrievedness,
      ) → ℝ;

    // Bias towards wanting the Good that is Food
    fn foodbias(
      self.nature.personality.openness, 
      self.stateful.currHunger) → ℝ;

    fn happiness(
      self.nature.personality.neuroticism,
      self.stateful.currAggrievedness,
      self.stateful.currExposure,
      self.stateful.currFatigue,
      self.stateful.currHunger,
      self.stateful.currThirst,
    ) → ℝ;
  }
  /** Events that befall an agent, dependent on their state */
  affects:
  {
    fn dieOfStarvation(self.currHunger) → Bool 
    { self.stateful.isAlive = false; }

    fn dieOfthirst(self.currThirs) → Bool
    { self.stateful.isAlive = false; }

    fn dieOfExposure(self.currExposure) → Bool
    { self.stateful.isAlive = false; }

    fn faintFromFatigue(self.currFatigue) → Bool
    { self.stateful.isAsleep = true; }
  }
  /** Events that an agent can bring about, dependent on their state */
  effects:
  {
    fn suicideFromMisery(self.composite.happiness) → Bool
    { self.stateful.isAlive = false; }
  }
...
}







```

```rs pesudocode
fn utility: fn(goods) → ℝ,
```


---

Where is this going?

- We already described the `goods` that exist, which is a `Set<Good>`.
  - Maybe we should be more precise and specify Resources, which are only `Good`s from the perspective of an `Agent`?
- We also arbitrarily chose one `Good` as a numeraire (hard-coded for now, in the future can vary by societ/ymarket/economy/clan/polity)
- Agents have
  - an aribitrary `self.nature` (immutable)
  - an arbitrary utility function over goods, which maps goods to a utility value (their preferences for each good)
    - `fn Utils(goods: Set<Good>, self.nature) -> Map<Good, ℝ>`
    - ==TODO/Question==: how do we ensure the utility function satisfies the axioms of utility?
- From an agent's utility function we can calulate `MU`, `MRS` of each good against the numeraire (money) good (let's say it's `Gold`) -- which is to say, we know the price they *would* pay for each good.
- Now, we can plot their multi-dimensional cobbs douglas preference curves. They only have their actual package of resources.

- Now that we know what they are indifferent to, we can determine what they would like to trade for.
  - We can generate possible buy/sell orders they could make and calculate the expected utility if those orders went through. If the expected utility of a trade is higher then their current utility, then we know that this would trade them towards a "higher" preference curve. They make the order that maximizes their utility.
    - I think we could so something similar in a bartering swap style AMM (either regular (e.g. Uniswap) or weighted (Balance)).
- And now that agents know what they would trade for, we can simulate a market and make a trading simulator.
  - ==TODO/Question==: is there a way to calculate all possible N-1 dimensional preference curves across N goods? If so, i think we could use this to generate optimal trading goals. This could be expecially interesting in a combinatorial auction.
- And once we have a trading simulator we can supervise, we can play as an Agent whose job it is to maximize our own utility, or else achieve any arbitrary aims (like maximizing gold while still being able to survive, or getting a monopoly on food to kill everyone else, or cause other market failures, or buy up all of the real estate, or buy up all of the "reputation" stock (political power)).