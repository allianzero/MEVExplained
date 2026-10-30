# MEVExplained

Maximal extractable value explained: frontrunning, sandwich attacks, and protection. MEV is the profit that can be extracted by controlling the order of transactions in a block — by reordering, inserting, or censoring them.

> This guide is educational only, not financial advice. MEV affects every user who trades on a public blockchain, whether they know it or not.

## 1. What MEV is, in plain language

When you submit a transaction to a public blockchain, it does not go straight into a block. It waits in a public waiting area called the **mempool**, where anyone can see it: what you are buying, how much, and your slippage tolerance.

The person or system that builds the next block — a miner in proof-of-work, a validator or block builder in proof-of-stake — chooses which transactions go in, and in what order. That ordering power has value, because the order of trades changes prices.

**Maximal extractable value (MEV)** is the maximum profit that can be extracted from that ordering power, beyond the normal block reward and fees.

Originally called "miner extractable value", the term changed to "maximal" when Ethereum moved to proof-of-stake and validators, builders, and searchers (not just miners) began extracting it.

## 2. Why ordering matters: a simple example

Imagine a liquidity pool with Token A priced at $1.00.

1. You submit a large buy order for Token A, visible in the mempool.
2. A bot sees your order, buys Token A first (pushing the price to $1.05), lets your order execute at the higher price, then sells its Token A at a profit.
3. You bought higher than you intended. The bot profited from your trade, without taking market risk — only ordering advantage.

This is not a hack of the blockchain. It is a consequence of public transactions plus ordering power. The chain worked exactly as designed; the design itself creates the opportunity.

## 3. The main MEV strategies

### Frontrunning
A bot sees your pending transaction and places its own transaction ahead of yours to profit from the price move your transaction will cause.

- Common target: large DEX trades, NFT mints, and liquidations.
- Defence: private transaction channels (see section 5), low slippage tolerance, and splitting large trades.

### Backrunning
A bot places a transaction immediately after yours to profit from the state change your transaction creates.

- Example: after a large trade moves a pool's price, a bot arbitrages the new price against another pool, before anyone else can.
- Backrunning is less directly harmful to the original trader than frontrunning, but it still extracts value that might otherwise have gone to ordinary users.

### Sandwich attacks
The combination: frontrun + your transaction + backrun, "sandwiching" your trade.

1. Bot buys before you, pushing the price up.
2. Your trade executes at the worse price.
3. Bot sells after you, capturing the difference.

Sandwich attacks are the most directly costly MEV for ordinary traders, and they are automated, constant, and invisible unless you check the price you actually received versus the price you expected.

### Arbitrage
Buying an asset cheaply on one DEX and selling it higher on another, in the same block or across blocks.

- Arbitrage itself helps prices stay aligned across markets — it is arguably useful MEV.
- But arbitrage bots compete fiercely, pay high gas to win ordering, and their profits come partly from ordinary traders' slippage.

### Liquidations
When a lending position becomes liquidatable, bots compete to execute the liquidation first and collect the liquidation bonus.

- This is useful MEV in the sense that liquidations keep lending protocols solvent.
- But liquidation races can also be manipulated: a bot that can move an oracle price or reorder transactions can trigger liquidations that would not otherwise have happened.

### Time-bandit / reorg attacks (advanced, rare)
A validator with enough power could rewrite recent blocks to capture MEV from past transactions. This is expensive and detectable on major chains, but it is the theoretical extreme of ordering power, and it is why finality and validator decentralisation matter.

## 4. Who extracts MEV, and who pays for it

The MEV supply chain, simplified:

- **Users** submit transactions. They pay the cost, usually as worse execution prices, without realising it.
- **Searchers** are bots and firms that find MEV opportunities in the mempool and craft profitable transaction bundles.
- **Builders** assemble blocks from searchers' bundles and users' transactions, choosing the most profitable ordering.
- **Validators / proposers** propose the block built by the builder they chose, and receive payment for it.
- **Protocols** sometimes capture MEV themselves (for example, through their own ordering or fee mechanisms) and redistribute it to users or token holders.

In short: ordinary users mostly pay MEV, sophisticated actors mostly collect it, and the split depends on the chain's design and the tools users employ.

## 5. How MEV affects you, concretely

- **Worse swap prices:** you receive less than the quoted price, especially on large trades in small pools.
- **Failed transactions that still cost gas:** your transaction is frontrun, the price moves beyond your slippage tolerance, the transaction fails — and you still pay gas.
- **NFT mint losses:** a bot frontruns a popular mint, buys the rare items, or pushes gas so high that ordinary minters cannot compete.
- **Liquidation risk:** in volatile markets, MEV bots can accelerate liquidations by ordering transactions strategically.
- **Hidden costs:** MEV is rarely itemised. It appears as "the price just moved" or "slippage was high", which makes it easy to underestimate.

## 6. Protection: what ordinary users can do

### Set slippage tolerance carefully
- **Too high:** you give sandwich bots a large profit window. A 5% tolerance on a large trade is an invitation.
- **Too low:** your transaction fails on normal price movement, and you pay gas for nothing.
- **Practical rule:** use the lowest tolerance that still executes reliably for that pair and trade size, and check the "minimum received" figure before confirming. For large trades, split them into smaller ones.

### Use private transaction channels
- **Private mempools / RPCs:** services (such as Flashbots Protect on Ethereum) submit your transaction directly to block builders without broadcasting it to the public mempool, so frontrunning bots cannot see it in advance.
- Trade-off: you trust the private channel operator, and your transaction may confirm slightly slower. For large or sensitive trades, this trade-off is usually worth it.

### Use MEV-aware DEXs and aggregators
- Some DEXs and aggregators route trades to minimise MEV, batch trades, or return MEV profits to users (for example, through auctions or rebates).
- Check whether your DEX documents MEV protection, and how it works, rather than assuming it.

### Split large trades
- Break a large swap into smaller pieces over time. Smaller trades move prices less and offer less profit to sandwich bots.
- Trade-off: more transactions mean more gas. Compare total gas against expected MEV savings.

### Avoid trading during extreme volatility if you can
- MEV extraction is most aggressive during crashes, launches, and liquidations, when price moves are largest. If a trade is not urgent, waiting for calmer conditions reduces both slippage and MEV.

### For NFT mints and launches
- Use the project's official mint, with a wallet containing only what you need for that mint.
- Expect gas wars and bot competition; decide your maximum gas in advance, and do not chase a mint beyond it.

## 7. Good MEV, bad MEV, and the debate

Not all MEV is harmful:

- **Arbitrage** keeps prices aligned across markets.
- **Liquidations** keep lending protocols solvent.
- **Backrunning** after a price move can be seen as efficient price discovery.

Harmful MEV is mainly:

- **Sandwich attacks and frontrunning** that directly worsen ordinary users' execution.
- **Censorship:** builders or validators excluding certain transactions (for example, for regulatory or competitive reasons).
- **Centralisation pressure:** MEV profits concentrate among sophisticated builders and validators, which can centralise block production over time.

The crypto community's response includes private mempools, MEV auctions that redistribute profits, encrypted mempools (where transaction contents are hidden until ordering is fixed), and protocol-level ordering rules. None of these fully solves MEV; each trades one problem for another.

## 8. Frequently asked questions

**Is MEV illegal?**
Generally, no — on most chains, ordering transactions is part of the protocol's design, and MEV extraction is not treated as illegal activity. Some forms, such as manipulating oracle prices to trigger liquidations, can cross into fraud or market manipulation under traditional law, but the legal status is unsettled and jurisdiction-dependent.

**Can I earn MEV as an ordinary user?**
Realistically, no, not by running a bot — that game is dominated by specialised firms with infrastructure advantages. Ordinary users "earn" MEV mainly by avoiding paying it: using private channels, sensible slippage, and MEV-aware routing.

**Does MEV exist on every chain?**
In some form, yes, wherever transactions are public before ordering and ordering affects prices. Its size and form vary: chains with private mempools, encrypted transactions, or centralised sequencers have different MEV profiles, not zero MEV.

**Is MEV the same as a scam?**
No. Scams deceive you into sending funds or approvals. MEV exploits public information and ordering rules that are part of the system. It is a design problem, not (usually) a deception — though the line blurs when MEV is used to manipulate markets.

**Should I worry about MEV for small trades?**
Less. Sandwich bots target trades large enough to profit from after gas costs. Small trades in deep pools lose little to MEV. Large trades in thin pools are where protection matters most.

> Educational content only. This is not financial, investment, or tax advice.
