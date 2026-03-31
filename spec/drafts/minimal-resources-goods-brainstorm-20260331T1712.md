
Simplifying assumptions and design heuristics:
- Each `Resource` maps to exactly one `Commodity`.
- `Commodity`s have multiple, non-overlapping uses so as to minimize the number of trading pairs and simplify the economy and simulation. Only commodities are tradeable. Commodities can be traded in CDAs
- Various ratios of `Commodity`s can be used to craft `Product`
- `Product`s can only be sold in English Auctions, or perhaps not at all


QN: which is simpler?
- Fewer commodities, many uses per commodity XOR many commodities, one use per commodity.

```rs

mapping Resource -> Commodity
  /** Bushes spawn manna at some per-bush rate.
  **Uses**:
  - Can be consumed to stave off hunger up to satiety
  - Can be consumed to restore HP up to full HP.  */
  | Bush -> Manna 

  /** Trees can be chopped down at some per-tree rate and yield per-tree wood.  */
  | Tree -> Wood 
  
  // Can be consumed (burnt) to stave off exposure, or crafted into buildings to make a permanent place that resets exposure

  | Rock -> 

  | MithrilOre -> MithrilMetal


```