
Simplifying assumptions and design heuristics:
- Each `Resource` maps to exactly one `Commodity`.
- `Commodity`s have multiple, non-overlapping uses so as to minimize the number of trading pairs and simplify the economy and simulation. Only commodities are tradeable. Commodities can be traded in CDAs
- Various ratios of `Commodity`s can be used to craft `Product`
- `Product`s can only be sold in English Auctions, or perhaps not at all


QN: which is simpler?
- Fewer commodities, many uses per commodity XOR many commodities, one use per commodity.


```ts
type Agent = {
  uuid: string,
  hunger: number,
  hp: number,
  exposure: number,
}

```

```rs

/**
- Resources can be *gathered*.
- Commodities are *held*, *used*, *consumed*, *traded* (in CDAs), and used as crafting ingredients for Products.
*/
mapping Resource -> Commodity
  /** Bushes spawn manna at some per-bush rate.
  **Manna Uses**:
  - can be consumed to stave off hunger up to satiety
  - can be consumed to restore HP up to full HP.  
  - can be used in crafting recipes
  */
  | Bush -> Manna 

  /** Trees can be chopped down at some per-tree rate and yield per-tree wood.
  **Wood Uses:**
  - can be consumed (burned) to stave off exposure
  - can be placed to create a block in a building (buildings reset exposure)
  - can be used in crafting recipes
  */
  | Tree -> Wood 
 
  /**
    Mithril Ore (non-renewable resource) can be mined at a global yield per rate with a per-deposit size.

    **Mithril Metal Uses:**
    - can be used as a numeraire (money) -- nominal value
    - can be used in advanced crafting recipes
  */
  | MithrilOre -> MithrilMetal


  /**
  Influence is a security that represents political power, kind of like stocks.

  **Influence Uses:**
  TBD
  */
  | Reputation -> Influence
```

```rs
/**
Products are *crafted from `Commodity`s*. They can be *held*, *used*, *consumed*, and *auctioned* (in English Auctions).
*/
mapping CraftingRecipes: CommodityBundle -> Product
| {Manna: 2, Time: 1} -> Bread
| {Wood: 3, Time: 1} -> Campfire
| {MithrilMetal: 1, Manna: 10, Time: 10} -> Sword
| {MithrilMetal: 1, Wood: 10, Time: 10} -> Shield
| {MithrilMetal: 1, Wood: 5, Time: 10} -> Pickaxe

```