# The All-Pay Auction: Modeling War, Attrition, and the Sunk-Cost Trap

Every other auction in this series has a comforting property: if you lose, you keep your money. The English auction, the sealed-bid auction, the CDA — losers walk away with their wallets intact.

The **All-Pay Auction** is different. Every bidder pays their bid. Only the highest bidder wins the prize. Everyone else loses both the prize *and* their money.

This sounds pathological. Why would anyone enter such an auction? And yet all-pay dynamics are everywhere in the real world — and they are exactly the right mechanism for modeling **war, conflict, R&D races, and territorial disputes** in your game.

---

## The Mechanism

1. A prize is announced (a territory, a resource node, a strategic advantage).
2. Agents commit resources — troops, gold, production capacity — as their "bids."
3. The agent that commits the most wins the prize.
4. **All agents lose what they committed**, win or lose.

The resources spent are gone. They're not refunded to losers. They're consumed by the act of competing itself — soldiers die, gold is spent on failed campaigns, R&D budgets are burned whether the patent lands or not.

---

## Why This Is the Right Model for War

Think about what happens when two kingdoms go to war over a gold mine.

Kingdom A commits 500 soldiers. Kingdom B commits 300 soldiers. Kingdom A wins the gold mine. But Kingdom B doesn't get its 300 soldiers back — they fought, they lost, many died. And Kingdom A doesn't get its 500 soldiers back unscathed either. Both sides paid. Only one side got the prize.

This is fundamentally different from a regular auction. In an English auction for the gold mine, Kingdom B would simply be outbid and walk away with its army intact. That doesn't model conflict. In an all-pay auction, the act of competing is itself costly and irreversible.

This maps to:
- **Military conflict:** Armies expend troops and resources regardless of outcome.
- **R&D races:** Multiple agents research the same technology. The first to complete it gets the advantage; the others wasted their research investment.
- **Political lobbying:** Factions spend influence to sway a ruling. Only one policy wins, but all factions burned their political capital.
- **Territory disputes:** Agents invest in fortifying and claiming a border region. The stronger claim wins, but both sides spent resources on the contest.

---

## The Dollar Auction: A Walkthrough of the Attrition Trap

The most famous illustration of all-pay dynamics is the **Dollar Auction**, devised by economist Martin Shubik. Let's run it in game terms.

**Setup:** The game master auctions a chest containing **100 Gold**. Rules: open ascending bids, highest bid wins the chest, but **both** the highest and second-highest bidders pay their final bids. Bids increment by 10 Gold.

**The Blacksmith and the Merchant both enter.**

| Round | Blacksmith's Bid | Merchant's Bid | Blacksmith's Position | Merchant's Position |
|---|---|---|---|---|
| 1 | 10 | — | If wins: +90 profit | — |
| 2 | — | 20 | Loses 10 if stops | If wins: +80 profit |
| 3 | 30 | — | If wins: +70 profit | Loses 20 if stops |
| ... | ... | ... | ... | ... |
| 9 | 90 | — | If wins: +10 profit | Loses 80 if stops |
| 10 | — | 100 | Loses 90 if stops | If wins: breaks even |

At this point, the Merchant has bid 100 Gold for a prize of 100 Gold — break-even at best. The rational move is to stop, right?

But look at the Blacksmith's position. If the Blacksmith stops now, it loses 90 Gold for nothing. If it bids **110**, it pays 110 for the 100 Gold chest — a net loss of 10. But that's *better* than the 90 Gold loss from stopping.

| Round | Blacksmith | Merchant | Logic |
|---|---|---|---|
| 11 | 110 | — | "Lose 10 is better than lose 90" |
| 12 | — | 120 | "Lose 20 is better than lose 100" |
| 13 | 130 | — | "Lose 30 is better than lose 110" |

**The bids spiral past the value of the prize.** Both agents are now bidding more than 100 Gold for a 100 Gold chest, because at every step, the incremental loss from bidding is smaller than the total loss from quitting. The sunk-cost fallacy and loss aversion lock them into an escalation spiral.

**The game master collects 130 + 120 = 250 Gold** for a 100 Gold prize. The all-pay structure extracts surplus far exceeding the prize value.

---

## The Game Theory: Mixed Strategy Equilibrium

If your agents are fully rational, they should recognize the trap before entering. The game-theoretic prediction for an all-pay auction with complete information is:

- There is **no pure-strategy Nash Equilibrium** (no fixed bid that is stable for all players).
- The equilibrium is in **mixed strategies**: agents randomize their bids according to a specific probability distribution.
- Under this equilibrium, every agent's **expected payoff is exactly zero**. The auctioneer extracts 100% of the surplus in expectation.

In practice — and this is key for your simulator — agents are rarely perfectly rational. They overcommit due to sunk costs, they escalate due to loss aversion, they misjudge opponents' willingness to continue. This means all-pay auctions in simulation tend to **over-extract** resources, creating dramatic, costly confrontations that deplete multiple agents simultaneously.

This is a *feature* for game design. Wars should be expensive. R&D races should drain treasuries. The all-pay auction naturally produces these dynamics.

---

## Design Patterns for Your Game

### Pattern 1: Military Conflict as All-Pay Auction

Two agents claim the same territory. Each secretly commits a number of troops. The agent with more troops wins the territory. Both armies suffer losses proportional to their commitment (the "payment").

```
// Simplified conflict resolution
fn resolve_conflict(attacker_troops: u32, defender_troops: u32) -> ConflictResult {
    let winner = if attacker_troops > defender_troops { Attacker } else { Defender };

    // Both sides pay — troops are consumed
    let attacker_losses = (attacker_troops as f64 * 0.7) as u32; // 70% casualties
    let defender_losses = (defender_troops as f64 * 0.6) as u32; // 60% casualties

    ConflictResult { winner, attacker_losses, defender_losses }
}
```

The losing side committed troops and got nothing. The winning side committed troops and gained territory, but at a real cost. This creates the strategic tension of all-pay dynamics: is the territory worth the troops you'll burn taking it?

### Pattern 2: R&D / Technology Races

Multiple agents invest gold into researching "Steel Forging." Each tick, agents choose how much to invest. The first agent to hit the cumulative investment threshold unlocks the technology. All other agents lose their partial investment.

This creates fascinating strategic dynamics:
- Agents must decide whether to enter the race at all (based on how many competitors they see).
- Once invested, they face the sunk-cost pressure to continue.
- An agent that sees a rival close to the threshold might either sprint to beat them or cut losses and abandon the race.

### Pattern 3: Lobbying / Political Influence

A council of elders will decide the tax rate. Each faction pays gold to lobby the council. The faction that spends the most gets its preferred policy enacted. All factions lose what they spent lobbying.

---

## Implementation Considerations

**Sealed vs. open format changes behavior dramatically.** In a sealed all-pay auction, agents commit simultaneously and can't observe opponents — this encourages the mixed-strategy equilibrium and avoids escalation spirals. In an open (ascending) format, agents see each other's commitments in real time, which enables the dollar-auction death spiral. Choose based on the game dynamic you want:
- Sealed: cleaner, more predictable, better for AI agents using expected-value calculations.
- Open: more dramatic, creates observable escalation, better for player-facing interactions.

**Budget constraints matter.** In the pure theory, the dollar auction spirals to infinity. In your game, agents have finite resources. An agent with 50 Gold can't bid 110. Budget constraints put a natural ceiling on escalation and make the game-theory analysis more tractable. They also create an asymmetric dynamic: a wealthy agent can bully poorer agents out of contests simply by signaling willingness to outspend them.

**Partial information creates bluffing.** If agents don't know each other's valuations or budgets, the all-pay auction becomes a game of bluff and inference. An agent might commit a large force to a low-value territory just to signal strength and deter future challenges. This is the game-theoretic analogue of a military deterrence posture.

**Revenue recycling.** In a game, the "revenue" from an all-pay auction — the resources all agents burned competing — doesn't go to an auctioneer. It's just *gone*. Troops died. Gold was spent on campaigns. This is a natural resource sink that prevents inflation and keeps the economy in check. If you want to soften the blow, you can have a fraction of the spent resources "recovered" by the winner (spoils of war) or redistributed as experience points.

---

## When to Use All-Pay Auctions in Your Game

Use all-pay mechanics whenever the act of competing is inherently costly and the cost cannot be recovered:

- **War and territorial conflict** — the defining use case. Both armies bleed.
- **Technology races** — R&D spending is sunk whether you win the patent or not.
- **Political contests** — lobbying burns resources regardless of outcome.
- **Resource extraction contests** — multiple agents mining the same deposit, spending effort and tool durability, with diminishing returns as they compete.

Don't use all-pay mechanics for ordinary commerce (use a CDA or AMM), for allocating items without inherent conflict (use a standard auction), or for any situation where losers should be able to walk away whole.

The all-pay auction is the mechanism that makes competition *hurt*. In a game with war and scarcity, that's exactly what you want. It forces agents to think carefully about which fights are worth entering — because every fight has a price, win or lose.
