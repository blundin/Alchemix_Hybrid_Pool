# Hybrid pools and the Transmuter

## Increasing capital efficiency

Getting deeper liquidity, and better returns for liquidity providers

Increasing capital efficiency—that is the main aim of all these defi products—allows people to do more with their money. This proposal introduces the hybrid pool  concept, explains what it is, what it may be able to achieve, and the potential drawbacks and risks. In a hybrid pool some portion of the funds are kept in another yield accrual protocol and then withdrawn and swapped out when trades are executed. This is used by the transmuter as an active balancing feature, maximising yield for debt repayment and contributing to the overall stability of the system.

### Terminology

- AL(ETH/USD/BTC/…) = ALC
- coin = C
  - where &quot;C&quot; can be any coin that Alchemix decides to add later.
- YC = yearn Coin (whenever some quantity of YC is counted its always the amount of YC \* the return ratio)
- arb = arbitrage
- LP = Liquidity Providers

### Life as we know it

The transmuter converts ALC to C. The purpose of the transmuter is to maintain the peg between ALC and C. The Curve LP pool provides liquidity between the ALC and C pair, so people can swap the less useful coin for the more useful one. Alchemix allows repayment in either ALC or C, so there is no reason to swap back into the ALC coin. This provides constant selling pressure on the ALC to C side, whcih creates a constant opportunity for arbitrage.

### Problems and inefficiencies with the current system

Constant selling pressure means someone has to arb the other way. Then they can transmute their ALC stick to convert it to C, and then keep on arbing back. This is a wasteful process that siphons funds out of the Alchemix ecosystem, reducing the yield for users and thus the transmuter. It also means that for most blocks on the Ethereum mainnet the LPers capital is not doing anything, this is again a waste.

### Good parts of the current system

The current system is more distributed and thus safer. The transmuter is not in the same smart contract as the ALC/C pool if something goes wrong in the ALC/C pool or the transmuter these are isolated and thus more protected. Combined contracts lead to a higher risk for security, bugs, or other issues. A reduced number of attack vectors leads to an overall safer environment.

### What is a hybrid pool?

The overarching aim of this hybrid pool is to increase the effciency of the invested capital, or put another way, increase the "working time" of the capital inside of the pool. One straightforward way to achieve this is by investing some portion of the assets in a Yearn vault and integrating that vault with the transmuter. So if the pool has (for example) a 5:90:5 split with the assets ALC:YC:C, the yield from the liquidity pool acts like additional deposits inside of the Alchemix ecosystem, increasing the yield and providing additional funds for the transmuter rebalancing. The number of coins inside of the YC section is measured as the amount of C that it holds, not the raw amount of YC, this means that as yield is generated for the LPers, the amount of liquidity inside of the system increases over time. To the user, they will just a see a 5:95 pool (ALC:C) and these mechanics will be abstracted away.

### Pool Mechanics

#### The transmuter

When the transmuter calls Yearn to draw down all funds and perform the rebalancing, it calls the Yearn rebalance function to withdraw/deposit YC/C from/to Yearn to the pool to keep the 90:5 (YC:C) ratio constant between them, then it sells YC and C into the pool to get it rebalanced between (ALC:YC:C).

#### Swaps

If the swap amount is over some threshold as a proportion of coins in the pool, then the YC are swapped back into C on their way out. This means that because we have larger liquidity from all the funds in the Yearn pool the whales (the people affected most by low liquidity) will be grateful for the lower slippage and wont mind the extra gas cost of drawing coins down from Yearn. The smaller fishes will not have to draw down from Yearn and instead will be paid out from the smaller 5% C pool.

#### Liquidity Providers

The pool works normally with the transmuter/Yearn draw down process acting as a stability mechanism on top. The LP tokens can be staked for ALCX rewards normally. The pool allows deposits normally and has all the features of a curve pool.

### The Scoopy is in the detail

This section is going to be a more detailed in-depth description of how the whole system could work.

The current system for reference:

![](https://raw.githubusercontent.com/biddls/Alchemix_Hybrid_Pool/main/LP%20cycle%20origional.png)

The hybrid pool for reference:

![](https://raw.githubusercontent.com/biddls/Alchemix_Hybrid_Pool/main/LP%20cycle%20new.png)

The best way to think of this may be to imagine the Curve pool sitting in the centre with all the additional addons floating around it on the outside, like the core curve pool is the same thing. Tacked on is the transmuter and a function to manage swaps in and out of the pool for the YC \&lt;-\&gt; C. The list of functions that are call able that are now either changed or new are:

- Swaps in and out of the pool
- LP additions
- Transmuter rebalance feature

#### Swaps in the pool work like so

Whale swap ALC-\&gt;C – if (slippage to C \&gt; slippage to YC + Yearn withdraw fee)

##### Swaps from ALC to YC

Withdraws YC from Yearn and receives C

Whale swap C -\&gt; ALC if (pool fee \&gt; gas cost to rebalance the pool (between YC and C))

Swaps from C to ALC with no pool fee (all the YC and C is counted as the same token, so we get that 5:95 ratio)

Then they pay for the rebalance of the pool, be that add or take out C from Yearn to reach that 90:5 ratio between them.

Shrimp swap ALC-\&gt;C

It&#39;s a normal Curve pool swap from ALC to C

Shrimp swap C -\&gt; ALC

It&#39;s a normal curve pool swap from C to ALC

#### LP additions

In normal operations the rebalancing between ALC:YC:C happens in the swaps

#### Transmuter rebalance

It calls the Yearn rebalance function, then it finds out how much YC and C it needs to sell into the pool to get the ratio back to normal; sells that much and burns the ALC it bought back.

### Its pegging time

The most important consideration that sits above all this; the overarching aim is keeping that peg. The measure of the peg is the price of ALC compared to C. So a value of 0 is on peg, 0.5 if 50% over C and -0.5 is 50% below C. if it&#39;s on peg is determined as such:

there are 3, technically 5 variables that go into an on or off peg scenario; the 5 variables are:

- a = how much ALC can the transmuter buy up as a % of coins in the pool
- a­1 = proportion of ALC that makes up the pool
- a2 = proportion of YC+C that makes up the pool (100% - a­1)
- t­1 = target proportion of ALC in the pool
- t2 = target proportion of YC+C in the pool (100% - t­1)

The red and green sections are the same curve shifted in the x axis by the amount of tokens that the transmuter can buy up because that&#39;s the balancing mechanism and so if it can buy up all those tokens then it can be easily made to be on &quot;peg&quot; allowing large amounts of selling on the ALC side and for it to still be able to keep its peg.

### Cucky Curve

Curve finance does not allow for people to modify their code so a new kind of LP pool will need to be devised. Building from the peg definition that was described earlier a UniSwap v2 pool and a constant price curve can be combined to provide 0 slippage over certain ranges and then the usual v2 Curve elsewhere. The graph below shows the same curve but with different scaling&#39;s, the one that looks like a stable swap curve is where the assets are 1:1 inside of the pool. The &quot;weird&quot; looking on represents a curve with a 1:5 ratio. The weird thing about this is that regardless of the ratios they will all sample a curve that looks like the 1:1 one. This is because the values will be scaled and normalised so that a 1:5 pool will behave like it is on a 1:1 curve.

![](on peg or na.png)

All of these examples simulate a pool with only 2 assets, this is still applicable as the YC/C coins are abstracted away to 1 coin in the background creating multiple curves but only showing one.

### Long term risk to the protocol that a hybrid pool helps to address

Currently we are &quot;paying&quot; people to provide liquidity to the protocol, and to do this we are using ALCX tokens. The issue arises that if ALCX does not outperform or match all assets we bring onto the platform then we run the risk of not being able to financially incentivise people to provide liquidity in the quantities that are required to do so. If ALCX falls by 50% we are now paying people 50% less for liquidity provision and thus it follows that there will be less liquidity for the system. If we have a hybrid pool then they will be earning returns on top of the LP fees at whatever yield source is being used % reducing the ALCX price risk.

### Summary

There seems to be a lot of positive reasons to implement this proposal, but it is a big project and would be a large undertaking, with many risks stemming just from the complexity of the project alone. I think I am to biased to really look critically at this proposal, so I now hand it over to you, the reader to message me @biddls.eth and roast the hell out of my idea. And hopefully after a nice bit of barbequing we will get a juicy proposal.

### What needs more development

- The 5:90:5 ratio needs more discussion; it could start out closer to a 3rd for each and then rebalance over time.
- When Alchemix is deployed on a L2 the pool could just be a ALC:YC pool, due to the lower fees.
