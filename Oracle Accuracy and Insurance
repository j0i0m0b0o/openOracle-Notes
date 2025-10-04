In openOracle, we say oracle accuracy is within +/- F where F ~= swap fees + jump loss + gas cost.

For example, say there's $100 of USDC and $100 of WETH in an oracle report instance with 1% swap fee and gas is $1 to both dispute and submit an initial report. The price of ETH is implied by the ratio of token amounts in the report.

For simplicity assume jump loss is zero. F here has the below components:

  1. 1% swap fee
  2. $1 of gas

In expectation a disputer earns the absolute difference in USD value of the two tokens at time of dispute and pays swap and gas fees which offsets the earnings. We will for now assume jump loss is zero and there are no other costs to reporting.

Here assume the price of ETH starts increasing. At +2%, there is $100 of USDC and $102 of WETH in the report instance. A new disputer waits until just over 2%, then disputes, earning the absolute difference (just over $2), pays $1 in swap fees (1% * $100 USDC), and pays $1 in gas. They then report a price that must be within roughly +/- 2% of the true price at time of dispute, or they would be instantly disputed by some other participant and lose money. Assuming the disputer does not have an external bet riding on the oracle outcome, the best price to report is generally the one closest to the true price at the time so you have a symmetric +/- 2% barrier around your price, reducing the likelihood of future disputes.

Because a new disputer reports when the true price is just past +/- F of the price at time of previous report, and then submits a new price that is within +/- F of the true price at dispute time, we say the oracle accuracy is within +/- 2%. In reality the accuracy improves based on the multiplier. For example, assume the oracle report instance has a multiplier of 1.25x. After the dispute there is $125 USDC and $125 WETH in the oracle instance and gas is still only $1.

The previous reporter only earns the 1% swap fee, so they receive about $1. The problem is, the price of ETH can go down too. Let's assume half the time it goes up and the previous reporter receives $1 in swap fees. The other half, the price goes down, and there is $100 of USDC and $98 in the report instance when a new disputer comes in. The previous reporter receives $0.98 in swap fees (1% * $98 WETH) since swap fees are in the current oracle code paid in the lower valued token. Changing the fees to be paid in the higher valued tokens may seem better in high gas fees at first glance but has tradeoffs which we will get into later.

Assume the previous reporter was the initial reporter, and earned some reward in ETH to submit $100 USDC and $100 of WETH to begin with.

Net flows sent in by report: $200 of tokens, $100 of USDC and $100 of WETH

Net flows sent back by dispute:

Case price goes up +2%: +$201 of tokens, $100 of USDC from initial report + $100 USDC from disputer + $1 swap fees from dispute
Case price goes down -2%: + ~$197 of tokens, $98 of weth from initial report + $98 of weth from disputer + $0.98 swap fees from dispute

If we assume price up and price down are each 50% likely, a new reporter loses $1 per report from this source of losses, assuming the price always moves enough for a dispute. So a new reporter must be compensated more before disputing, widening oracle accuracy past +/- 2%

It may seem like this a blowup point for the oracle game because this $1 loss widens the dispute band from +/- 2% to +/- 3% which increases the $1 loss to $2 which further widens the band to +/- 4% and so on. But in reality, there are several mitigating factors here:

1. The price doesn't always move enough for a dispute to fire (dispute probability is not 100%)
2. A previous disputer can self-dispute which results in tighter dispute bands. A new disputer also knows they have the self-dispute move to play later if necessary.
3. The wider the dispute band, the lower the dispute probability, strictly, and so the lower the cost to an honest reporter. The dispute income from the absolute difference in values strictly increases with a wider dispute band as well. So the oracle accuracy finds an equilibrium past +/- 2%.
4. When dispute probability is high, the amounts in the oracle game escalate exponentially in proportion to dispute frequency which reduces the impact of gas fees on reporter profitability geometrically.

Before we get into insurance contracts that are designed to mitigate high gas scenarios, let's see how the game plays out if gas is $0, using contract-equivalent logic for the minimum dispute band economics.

So we start again with $100 USDC and $100 WETH. Since gas is 0, the disputer is willing to dispute when the price of WETH goes up or down 1%. Before, we were considering +/- 2% which are additive barriers just out of simplicity. From a contract-equivalent perspective, the oracle contract prevents disputes inside the previous report's fee band. It is computed as follows: the upper barrier is 1.01 * old_price, and the lower barrier is old_price / 1.01. 

Said simpler the dispute up can happen past +1% and the dispute down can happen past -0.990099% (1 - 1/1.01)

Because a disputer pays the 1% swap fees on the lower valued token, they are willing to dispute just past -0.99% instead of -1%, since their costs decreased with the value of the token. In fact the disputer's breakeven lower barrier is exactly the contract-equivalent lower barrier assuming 0 gas fees and jump loss. So paying fees in the lower valued token gives us a nice accuracy range that is symmetric in log space, which is how real world prices actually act.

So if we do the math again on previous reporter profitability with zero gas fee:

Net flows sent in by report: $200 of tokens, $100 of USDC and $100 of WETH

Net flows sent back by dispute ($0 gas fee):

Case price goes up +1%: +$201 of tokens, $100 of USDC from initial report + $100 USDC from disputer + $1 swap fees from dispute
Case price goes down ~ -0.99%: exactly +$199 of tokens, ~$99.00990099 of weth from initial report + ~$99.00990099 of weth from disputer + ~ $0.9900990099 swap fees from dispute

If we assume price up +1% and price down (1 - 1/1.01) are each 50% likely, a new reporter loses exactly 0 in expectation from this game. So we can see things are much cleaner with low or neglibile gas.

Since baseFee is accessibly by solidity, applications can set the initial oracle liquidity in relation to prevailing gas fees to minimize loss in accuracy from excess gas fees, at time of price request. Assume out of conservatism you use 4x the oracle liquidity you actually need relative to the prevailing gas fees. The longer the game goes on, the more disconnected the prevailing prices (including priority fees) can become relative to where they were at oracle instance creation. For example gas fees can spike after the initial report and oracle accuracy can decrease nonlinearly.

If a lot of time passes before the gas spike, the oracle instance is guaranteed to have escalated exponentially quickly which moderates things even with extreme gas spikes. The real risk is if gas prices spike at time of initial report or in the first few disputes. There is another issue for standing liquidity pools taking bets against anyone-can-create oracle price requests in which an attacker could wait until right before a large NFT mint and try to bet against the pool using an oracle instance that had an initial report right before the NFT mint.

In theory, if we paid people to dispute reports when gas spiked, we could escalate the oracle game to where accuracy is not impacted adversely by gas spikes.

We will ignore block producers manipulating the baseFee for now. Imagine an insurance smart contract which contains some large ETH balance. Anyone who creates a report instance in the oracle can optionally pay for insurance on this reportId covered by the insurance pool. The baseFee at time of report instance creation is recorded in the insurance contract. The insurer can specify some multiplier, say 4x, where if the prevailing baseFee rises 4x relative to what was recorded, someone can get paid some amount to call a function which gives a start block for an exponentially increasing bounty to call disputeAndSwap on the covered reportId. Once disputed, the bounty and start block are reset and exponentially escalate again, until the report instance has been sufficiently escalated. 

Adverse selection is a very big challenge for an insurance smart contract that covers any report instance from anyone, even if the extractable profit is eaten up by gas and then MEV on top. So for now just assume the application that wants insurance maintains their own insurance pool, or the insurance contract is highly limited to certain parameter sets.

The higher the multiplier, the lower the insurance costs. For example, let's assume you want protection against a 100x gas spike. Before we said out of conservatism, you set the oracle initial liquidity 4x too high relative to prevailing basefee. so you need to escalate the oracle amounts by 25x for oracle accuracy to be back to acceptable levels. At a multiplier of 1.5 this is 8 disputes, or 8 dispute gas costs paid by the insurance contract. In the case the underlying price that the oracle is reporting on doesn't actually move, the insurance contract payout escalates until gas and jump loss is covered to make a same price dispute worth it. But since gas is so high, jump loss (which only accrues from disputes where the true price is outside the fee barrier) is really low. It is very worth it for a disputer to take the bounty and escalate the game early on which limits the damage to the insurers.

If a insurance buyer cannot contrive excessive insurance payouts, even if the 8 disputes cost 800 times the dispute gas costs at time of insurance purchase, the insurance premium should remain low and a function of starting dispute gas cost.

For example, let's imagine a hypothetical lending contract similar to AAVE that lets someone create a liquidation price request in openOracle, where a liquidation initiator must provide a liquidation bet in proportion to the max equity up for grabs in a given lending position that is liquidatable. If the oracle comes back with a price that does not liquidate the specified position, the initiator loses their bet and nobody is liquidated. As an aside it is conceivable to have the liquidation initiator pay a large amount for the openOracle initial reporter to ensure a lot of initial liquidtiy is there to liquidate with. A manipulator would try to wait until gas spikes or was just about to spike, then try to initiate a liquidation and manipulate the oracle price and make more from the liquidation than was paid in gas fees submitting bad prices to the oracle. Since gas is too high, nobody would dispute the report and the price would settle badly, liquidating an assumed-passive lender. The lender could step in and escalate the oracle themselves and lose only the gas fees, and it is possible it is net worth it to do this if the liquidation initiator's bet goes directly to the lender in case of no liquidation, but in general setups like this that require people to pay attention and not be passive are costly and we should try to avoid them. If we allow the settled oracle price to liquidate any lending position less than or equal to the size of the targeted position, this does increase the number of people for whom it is worth it to dispute the oracle and it is 1 of n to choose the less EV- outcome and eat the gas costs to avoid liquidation penalty, even if only the targeted position receives the bounty.

But we are trying to use insurance contracts here even if there is some stable game theoretical route with all active participants. With the insurance contract, let's assume there is only one price request at a given time that can be used across all positions to liquidate, but the initiator still must target a specific position and only earns a liquidation reward from that position (they would choose the most valuable one). Let's say the initiator knows some NFT mint is coming in 10 seconds. they initiate a liquidation of some position. Let's assume the lending contract uses max(floor liquidity, some fraction of targeted lending position) as the initial oracle liquidity. The insurance contract is forced to pay out say 8 disputes at 100x gas cost, but this only happens once, and the oracle price still settles correctly. If the manipulator tries again instantly, the prevailing base fee will have risen and they would need another predictable increase from there to damage the pool. If the insurance pool receives part of the liquidation rewards too alongside insurance premiums, since liquidation rewards scale with capital and the dispute costs are more or less fixed, the insurance pool should have enough money to ensure gas costs do not reduce oracle accuracy almost ever.
