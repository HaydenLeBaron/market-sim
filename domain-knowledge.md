# Market & Auction Types: Domain Knowledge Reference

This document provides an in-depth reference for each market and auction mechanism that `market-sim` aims to support. Every section follows the same structure: definition, mechanism, real-world examples, key economic properties, and a concrete walkthrough.

For foundational terminology, see the [Glossary](#glossary) at the end.

---

## 1. Traditional Bartering

### Definition

Bartering is the direct exchange of goods or services between two parties without using money or any other medium of exchange. It is the oldest form of trade and requires a **double coincidence of wants**: each party must possess something the other desires.

### Mechanism

1. Two parties each hold goods or services.
2. They discover that each wants what the other has.
3. They negotiate an exchange ratio (e.g., "3 wood for 1 iron").
4. The swap occurs simultaneously -- there is no auctioneer, no order book, and no price system denominated in a common unit.

### Real-World Examples

- **Pre-monetary economies:** Ancient communities trading grain for livestock.
- **Modern barter networks:** Platforms like ITEX facilitate cashless business-to-business exchanges.
- **International countertrade:** Countries with limited foreign currency reserves trade oil for machinery.
- **In-game item trading:** Players swap inventory items directly (e.g., 5 potions for a sword).

### Key Properties

| Property | Value |
|---|---|
| Incentive compatible? | No formal guarantee -- depends entirely on negotiation |
| Efficiency | Low. With *N* goods there are *N(N-1)/2* possible exchange rates to track |
| Information revealed | Only what the two parties disclose to each other |
| Scalability | Poor -- the double coincidence of wants becomes harder to satisfy as the economy grows |

The inefficiency of barter is precisely what motivates the introduction of **money** (a numeraire) and structured market mechanisms.

### Walkthrough Example

**Setup:** Alice has 10 apples and wants fish. Bob has 8 fish and wants apples.

| | Apples | Fish |
|---|---|---|
| **Alice (before)** | 10 | 0 |
| **Bob (before)** | 0 | 8 |

**Negotiation:** They agree on a rate of 2 apples per fish. Alice wants 3 fish, so she offers 6 apples.

**After the trade:**

| | Apples | Fish |
|---|---|---|
| **Alice (after)** | 4 | 3 |
| **Bob (after)** | 6 | 5 |

Both parties are better off by their own assessment -- this is a **Pareto improvement**. But note: if Alice also wanted iron, she would need to find a *third* party willing to trade iron for something she has. This coordination problem is why structured markets exist.

---

## 2. English (Ascending) Auction

### Definition

An English auction is an open-cry, ascending-price auction for a single item. The price starts low and rises as bidders compete. The last bidder standing wins and pays their final bid.

### Mechanism

1. The auctioneer announces a starting price (often a **reserve price** below which the item will not sell).
2. Bidders openly place increasingly higher bids.
3. Each new bid must exceed the current highest bid (often by a minimum increment).
4. When no bidder is willing to go higher, the auctioneer closes the auction.
5. The highest bidder wins and pays the amount of their final bid.

### Real-World Examples

- **Art auctions:** Sotheby's and Christie's use English auctions for paintings and collectibles.
- **eBay:** Proxy bidding on eBay approximates an English auction -- the platform auto-bids up to your maximum on your behalf.
- **Spectrum auctions:** Governments sell wireless spectrum licenses using ascending-clock variants.

### Key Properties

| Property | Value |
|---|---|
| Incentive compatible? | Weakly yes -- the dominant strategy is to stay in the bidding until the price reaches your true valuation, then drop out |
| Strategic equivalence | Equivalent to the Second-Price (Vickrey) Sealed-Bid auction under independent private values (IPV) |
| Information revealed | High -- all bids are public, so participants learn about others' valuations in real time |
| Winner's curse | Mitigated (you can see how aggressively others are bidding) |
| Revenue equivalence | Under IPV with risk-neutral bidders, expected revenue equals that of the other standard auction formats |

### Walkthrough Example

**Setup:** A painting is up for auction with a reserve price of $100. Three bidders have private valuations:

- Alice values the painting at $500
- Bob values it at $350
- Carol values it at $200

**Bidding sequence:**

| Round | Bidder | Bid |
|---|---|---|
| 1 | Alice | $100 |
| 2 | Bob | $150 |
| 3 | Carol | $180 |
| 4 | Alice | $200 |
| 5 | Bob | $220 |
| 6 | *(Carol drops out -- price exceeds her $200 valuation)* | |
| 7 | Alice | $250 |
| 8 | Bob | $300 |
| 9 | Alice | $350 |
| 10 | Bob | $360 |
| 11 | Alice | $370 |
| 12 | *(Bob drops out -- price exceeds his $350 valuation)* | |

**Result:** Alice wins at **$370**. Notice she pays well below her $500 valuation. The final price is just above Bob's valuation ($350), not Alice's. This mirrors the second-price property: the winner effectively pays a price determined by the *second-highest* valuer.

---

## 3. Dutch (Descending) Auction

### Definition

A Dutch auction starts at a high price that descends over time. The first bidder to accept the current price wins the item and pays that price. It is called "Dutch" because of its historical use in the Netherlands for selling flowers.

### Mechanism

1. The auctioneer (or a clock display) starts at a price well above the expected market value.
2. The price decreases at regular intervals (e.g., by $50 every few seconds).
3. The first participant to call out "Mine!" (or press a button) wins.
4. The winner pays the price at the moment they claimed the item.
5. No other bids are placed -- the auction is over in a single action.

### Real-World Examples

- **Aalsmeer Flower Auction (Netherlands):** The world's largest flower auction, processing ~20 million flowers per day using descending clocks.
- **US Treasury bills:** The Treasury uses a modified Dutch auction format for bill sales.
- **IPOs:** Google's 2004 IPO used a Dutch auction variant to set its share price.

### Key Properties

| Property | Value |
|---|---|
| Incentive compatible? | No -- the optimal strategy is to shade your bid below your true valuation (wait for the price to drop further), accepting the risk that someone else claims the item first |
| Strategic equivalence | Equivalent to the First-Price Sealed-Bid auction |
| Information revealed | None during the auction -- no one sees any bids until the single winning bid occurs |
| Speed | Very fast -- resolves in a single decision point |

### Walkthrough Example

**Setup:** A crate of tulips is for sale. The clock starts at $1,000 and decreases by $50 every tick.

- Alice values the crate at $700
- Bob values it at $500

**Sequence:**

| Tick | Price | Event |
|---|---|---|
| 1 | $1,000 | No action |
| 2 | $950 | No action |
| 3 | $900 | No action |
| 4 | $850 | No action |
| 5 | $800 | No action |
| 6 | $750 | No action |
| 7 | $700 | Alice claims! |

**Result:** Alice wins at **$700**. She chose not to wait further -- if she had tried to get a lower price (say $650), she risked Bob jumping in at $550 or $500. The strategic tension is between "get a better deal" and "lose the item entirely."

Notice: if Alice had bid optimally by shading slightly (e.g., waiting for $650), she would capture more surplus -- but she would also risk losing the item. This risk-reward tradeoff is identical to the one in a first-price sealed-bid auction.

---

## 4. First-Price Sealed-Bid Auction

### Definition

In a first-price sealed-bid auction, every bidder independently submits a single secret bid. The highest bidder wins and pays exactly the amount they bid.

### Mechanism

1. The auctioneer announces the item and rules.
2. Each bidder submits a sealed bid without seeing others' bids.
3. All bids are opened simultaneously.
4. The highest bidder wins and pays the amount of their own bid.

### Real-World Examples

- **Government procurement:** Contractors submit sealed bids for public infrastructure projects.
- **Real estate:** "Best and final offer" situations where buyers submit blind offers.
- **Oil and gas lease auctions:** Energy companies bid on drilling rights.

### Key Properties

| Property | Value |
|---|---|
| Incentive compatible? | No -- rational bidders shade their bids below their true valuations |
| Strategic equivalence | Equivalent to the Dutch (Descending) auction |
| Dominant strategy? | None -- the optimal bid depends on beliefs about others' valuations and their bidding strategies |
| Information revealed | None before or during bidding; bids may or may not be revealed after |
| Revenue equivalence | Under IPV with risk-neutral bidders, expected revenue matches other standard formats |

**Why shade your bid?** If your true value is $100 and you bid $100, you win but earn zero surplus. If you bid $85, you might still win (if no one else bids above $85) and keep $15 in surplus. The optimal shade depends on how many bidders there are and what you believe about their valuations.

### Walkthrough Example

**Setup:** Three bidders compete for a government contract. Their private valuations (maximum willingness to pay):

- Alice: $100
- Bob: $80
- Carol: $60

Suppose each bidder shades their bid to roughly the expected second-highest valuation among competitors (a common equilibrium strategy with uniformly distributed values):

| Bidder | True Value | Submitted Bid |
|---|---|---|
| Alice | $100 | $82 |
| Bob | $80 | $65 |
| Carol | $60 | $48 |

**Result:** Alice wins and pays **$82**. Her surplus is $100 - $82 = $18.

Compare to an English auction where Alice would have won at ~$80 (Bob's dropout point). In this first-price auction, Alice may pay slightly more or less than $80 depending on how aggressively everyone shades. On average, revenue equivalence holds -- the auctioneer's expected revenue is the same across formats under standard assumptions.

---

## 5. Second-Price (Vickrey) Sealed-Bid Auction

### Definition

A Vickrey auction is a sealed-bid auction where the highest bidder wins but pays the **second-highest** bid, not their own. Named after economist William Vickrey (Nobel Prize, 1996), it is one of the most celebrated mechanisms in auction theory because of its elegant incentive properties.

### Mechanism

1. Each bidder submits a sealed bid independently.
2. All bids are opened simultaneously.
3. The highest bidder wins.
4. The winner pays the amount of the **second-highest** bid.

### Real-World Examples

- **Online advertising:** Google Ads uses a Generalized Second-Price (GSP) auction for ad placement -- not exactly Vickrey, but closely inspired by it.
- **Stamp auctions:** Some philatelic auctions use Vickrey rules.
- **Theoretical benchmark:** The Vickrey auction is the canonical example of a truthful mechanism in mechanism design theory.

### Key Properties

| Property | Value |
|---|---|
| Incentive compatible? | **Yes** -- bidding your true valuation is a **weakly dominant strategy** |
| Dominant strategy | Bid your true value |
| Information revealed | Bids are sealed; only the winning bid and second-highest bid may be announced |
| Revenue equivalence | Under IPV with risk-neutral bidders, expected revenue equals that of the English auction |

**Why is truthful bidding dominant?** Consider two deviations:

- **Overbidding** (bidding above your value): You might win auctions where the second-highest bid exceeds your true value, meaning you pay more than the item is worth to you. You lose money.
- **Underbidding** (bidding below your value): You might lose auctions where the second-highest bid is between your bid and your true value. You miss profitable trades.

Neither deviation can improve your outcome. Bidding truthfully is always at least as good as any other strategy.

### Walkthrough Example

**Setup:** Three bidders, each with a private valuation:

- Alice: $100
- Bob: $80
- Carol: $60

Since truthful bidding is dominant, each bids their true value:

| Bidder | True Value | Bid |
|---|---|---|
| Alice | $100 | $100 |
| Bob | $80 | $80 |
| Carol | $60 | $60 |

**Result:** Alice wins. She pays Bob's bid: **$80** (the second-highest).

Alice's surplus: $100 - $80 = **$20**.

Notice this is approximately the same outcome as the English auction walkthrough, where Alice won at $370 (just above Bob's $350 valuation). The Vickrey mechanism achieves the same result without the back-and-forth bidding rounds -- one sealed bid from each participant suffices.

---

## 6. Call (Batch) Double Auction

### Definition

A call auction (also called a batch auction or clearing auction) collects buy orders (bids) and sell orders (asks) over a defined window of time, then executes all matched trades simultaneously at a single **clearing price**. Unlike single-item auctions, a double auction involves both buyers and sellers.

### Mechanism

1. During the collection window, buyers submit bids (price and quantity they are willing to buy at) and sellers submit asks (price and quantity they are willing to sell at).
2. When the window closes, the auctioneer aggregates all orders:
   - **Demand curve:** Sort bids from highest to lowest price. Cumulative quantity at each price gives aggregate demand.
   - **Supply curve:** Sort asks from lowest to highest price. Cumulative quantity at each price gives aggregate supply.
3. The **clearing price** is the price at which aggregate demand equals aggregate supply (or the midpoint of the range where they cross).
4. All bids at or above the clearing price are matched with all asks at or below the clearing price.
5. Every matched trade executes at the single clearing price.

### Real-World Examples

- **Stock exchange opening/closing auctions:** The NYSE and LSE use call auctions to set opening and closing prices each day, aggregating overnight or end-of-day order flow.
- **Electricity markets:** Day-ahead power auctions determine the next day's electricity prices by matching generators' supply offers with utilities' demand bids.
- **Experimental economics:** Vernon Smith's pioneering market experiments used call auctions to demonstrate convergence to competitive equilibrium.

### Key Properties

| Property | Value |
|---|---|
| Incentive compatible? | Approximately -- large call auctions with many participants approach incentive compatibility, but individual participants in small auctions can benefit from strategic bid shading |
| Price discovery | Aggregated -- all information is pooled before any trades occur |
| Manipulation resistance | Higher than continuous trading -- manipulators cannot exploit the sequence of individual trades |
| Uniform pricing | All trades execute at one price, which is considered "fair" in the sense that no buyer pays more or seller receives less than the market-clearing rate |

### Walkthrough Example

**Setup:** A call auction for iron ingots in a game economy. Collection window closes with these orders:

**Buy orders (bids):**

| Buyer | Bid Price | Quantity |
|---|---|---|
| Alice | $52 | 100 |
| Bob | $50 | 200 |
| Carol | $48 | 150 |

**Sell orders (asks):**

| Seller | Ask Price | Quantity |
|---|---|---|
| Dave | $47 | 150 |
| Eve | $49 | 100 |
| Frank | $51 | 200 |

**Constructing the curves:**

Demand (cumulative from highest bid):
- At $52: 100 units demanded (Alice)
- At $50: 300 units demanded (Alice + Bob)
- At $48: 450 units demanded (Alice + Bob + Carol)

Supply (cumulative from lowest ask):
- At $47: 150 units supplied (Dave)
- At $49: 250 units supplied (Dave + Eve)
- At $51: 450 units supplied (Dave + Eve + Frank)

The curves cross between $49 and $50 -- at $50, demand is 300 and supply is 250. At $51, supply jumps to 450 and demand drops to 100. The clearing price is set at **$50**.

**Matched trades at $50:**
- Dave sells 150 at $50 (asked $47 -- gets more than his minimum)
- Eve sells 100 at $50 (asked $49 -- gets more than her minimum)
- Alice buys 100 at $50 (bid $52 -- pays less than her maximum)
- Bob buys 150 at $50 (bid $50 -- pays exactly his maximum)

**Unmatched:** Carol (bid $48, below clearing price) and Frank (asked $51, above clearing price) do not trade.

Total volume: **250 units** traded at **$50**.

---

## 7. Continuous Double Auction (CDA)

### Definition

A continuous double auction is an ongoing market where buyers and sellers can submit orders at any time. Whenever a buyer's bid meets or exceeds a seller's ask, a trade executes immediately. This is the mechanism underlying most modern financial exchanges.

### Mechanism

1. The market maintains an **order book** with two sides:
   - **Bid side:** Buy orders sorted by price (highest first). At equal prices, earlier orders have priority (price-time priority).
   - **Ask side:** Sell orders sorted by price (lowest first). Same time-priority rule.
2. When a new order arrives:
   - If it's a **bid** that is >= the best (lowest) ask, a trade executes immediately at the ask price (the resting order's price).
   - If it's an **ask** that is <= the best (highest) bid, a trade executes at the bid price.
   - Otherwise, the order is added to the book as a **resting limit order**.
3. **Market orders** (orders with no price limit) execute immediately against the best available resting order.
4. The **spread** is the gap between the best bid and best ask. A narrower spread indicates a more liquid market.

### Real-World Examples

- **Stock exchanges:** NASDAQ, NYSE (continuous session), Tokyo Stock Exchange -- virtually all modern equity trading uses a CDA.
- **Foreign exchange (Forex):** The ~$7 trillion/day forex market operates as a decentralized CDA.
- **Cryptocurrency exchanges:** Binance, Coinbase, Kraken -- all use CDA order books.

### Key Properties

| Property | Value |
|---|---|
| Incentive compatible? | No general incentive compatibility -- strategic order placement (timing, limit vs. market) is crucial |
| Price discovery | Continuous -- prices update in real time as new information arrives |
| Information revealed | High -- the full order book (all resting bids and asks) is typically visible |
| Convergence | Vernon Smith (Nobel 2002) showed that CDAs converge to competitive equilibrium even with few, poorly informed traders |
| Liquidity | Measured by the bid-ask spread and order book depth |

### Walkthrough Example

**Setup:** A CDA for wood planks. The order book starts empty.

**Step 1:** Alice places a **bid** of $49 for 10 units.

| Bid Side | Ask Side |
|---|---|
| Alice: $49 x 10 | *(empty)* |

No trade -- no asks to match against.

**Step 2:** Bob places an **ask** of $51 for 10 units.

| Bid Side | Ask Side |
|---|---|
| Alice: $49 x 10 | Bob: $51 x 10 |

No trade -- best bid ($49) < best ask ($51). Spread = $2.

**Step 3:** Carol places a **bid** of $51 for 5 units.

Carol's bid ($51) >= Bob's ask ($51). **Trade executes:** 5 units at $51.

| Bid Side | Ask Side |
|---|---|
| Alice: $49 x 10 | Bob: $51 x 5 *(remaining)* |

**Step 4:** Dave places an **ask** of $48 for 8 units.

Dave's ask ($48) <= Alice's bid ($49). **Trade executes:** 8 units at $49.

| Bid Side | Ask Side |
|---|---|
| Alice: $49 x 2 *(remaining)* | Bob: $51 x 5 |

**Step 5:** Eve places an **ask** of $50 for 6 units.

Eve's ask ($50) > Alice's bid ($49). No trade. The order rests on the book.

| Bid Side | Ask Side |
|---|---|
| Alice: $49 x 2 | Eve: $50 x 6 |
| | Bob: $51 x 5 |

Spread is now $1 ($49 best bid, $50 best ask). The order book has become more liquid.

**Trade log:**

| Time | Buyer | Seller | Price | Quantity |
|---|---|---|---|---|
| Step 3 | Carol | Bob | $51 | 5 |
| Step 4 | Alice | Dave | $49 | 8 |

---

## Comparison Table

| Mechanism | Incentive Compatible? | Price Discovery | Speed | Information Revealed During |
|---|---|---|---|---|
| **Bartering** | No guarantee | Bilateral negotiation | Slow (requires finding a match) | Only between the two parties |
| **English Auction** | Weakly yes | Ascending, real-time | Moderate (multiple rounds) | High (all bids are public) |
| **Dutch Auction** | No (shade bids) | None until resolution | Very fast (single action) | None |
| **First-Price Sealed-Bid** | No (shade bids) | None until resolution | Fast (single round) | None |
| **Second-Price (Vickrey)** | **Yes** (dominant strategy) | None until resolution | Fast (single round) | Minimal (winner & price) |
| **Call Double Auction** | Approximately (large markets) | Aggregated batch | Periodic | All orders after clearing |
| **Continuous Double Auction** | No | Continuous, real-time | Immediate (on match) | High (order book visible) |

---

## Glossary

- **Ask:** An offer to sell at a specified price (also called an "offer").
- **Bid:** An offer to buy at a specified price.
- **Bid-ask spread:** The difference between the best (highest) bid and the best (lowest) ask in an order book. A measure of liquidity.
- **Clearing price:** The single price at which all trades execute in a call/batch auction.
- **Dominant strategy:** A strategy that is optimal for a player regardless of what other players do.
- **Double coincidence of wants:** The situation where two parties each have what the other wants -- a prerequisite for barter.
- **Incentive compatibility:** A mechanism is incentive-compatible if every participant's best strategy is to act according to their true preferences (e.g., bid their true valuation).
- **Independent private values (IPV):** A model where each bidder's valuation is drawn independently and privately, and knowing others' valuations would not change one's own.
- **Limit order:** An order to buy or sell at a specific price (or better). It rests on the order book until matched or cancelled.
- **Market order:** An order to buy or sell immediately at the best available price.
- **Numeraire:** A standard unit of account (money) used to express prices, eliminating the need for pairwise exchange rates.
- **Order book:** The collection of all outstanding bids and asks in a continuous double auction.
- **Pareto improvement:** A change that makes at least one party better off without making anyone worse off.
- **Reserve price:** The minimum price an auctioneer will accept. If no bid meets the reserve, the item is not sold.
- **Revenue equivalence theorem:** Under standard assumptions (IPV, risk-neutral bidders, symmetric distributions), all standard auction formats yield the same expected revenue for the seller.
- **Vickrey auction:** A second-price sealed-bid auction. Named after William Vickrey.
- **Winner's curse:** The phenomenon where the winner of an auction tends to have overpaid, particularly in common-value settings where the item's value is the same for all bidders but uncertain.
