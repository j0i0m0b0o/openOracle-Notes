Based on the underlying market price action and reasonable assumptions about dispute network participation, exponential escalation can shift future costs back into the current oracle accuracy.

More specifically, if we assume a disputer is looking at some report, and then there is a worst-case self-dispute every round after that with some proportional cost (like a % burn fee or jump loss), and we have a multiplier M and a per-round settlement (no-dispute) probability Q, then we require

> **Q > 1 − 1/M**

for the expected total future cost of that “always self-dispute” strategy to stay finite. If actual Q makes this inequality false, the expected future cost can become arbitrarily large, so a rational trader may refuse to dispute unless the mispricing is very big — effectively widening the no-dispute band and worsening oracle accuracy.

The derivation can be found more or less [here](https://github.com/j0i0m0b0o/openOracle-Notes/blob/main/old%20docs%20post%20on%20multiplier%20threshold).

This is an unrealistic scenario because it assumes there are no other disputers but somehow self-disputes are still needed. It also ignores the “dispute and refuse to self-dispute” path, which makes oracle accuracy widen from ±F to roughly ±(F + M × expected move over the settlement time), assuming at least one other disputer pursuing the same strategy. Here F can be thought of as the worst-case no-dispute band from fees; in this simplified model we can roughly take F ≈ swap fees + 2 × protocol fees.

There is another important self-regulating step. As the no-dispute band grows, Q increases mechanically, so this moderates the apparent severity of crazy distributions.

But it is a helpful framing to understand the costs and move toward a more refined oracle game design for time-critical report IDs like liquidations:
	
	1.	Swap-fee-only up to escalation halt, at which point burn kicks in (M = 1 here makes this distributionally nice since required Q > 0 structurally).
	
	2.	Self-disputes prior to escalation halt don’t pay any proportional fee since self-disputers can get a dispute in before it is profitable for the rest of the network.
	
	3.	Alternative escalation that lets anyone escalate up to this amount to take advantage of a delay attacker’s willingness to pay swap fees.
	
	4.	This allows for higher multipliers without importing an ugly tail into current oracle accuracy. The cost of capital is the main cost transferred from future disputes to the current oracle accuracy in the worst-case self-dispute game.

An alternative to 1) above is disputeDelay-only up to escalation halt. This imports the market's expected move into oracle accuracy. If the band were ever larger than the expected move, self-disputes are free, so the band naturally tightens. It also recalibrates every round - and the survival probability inside +/- 1 expected move is relatively robust. This mechanically stabilizes Q by itself which cuts off the nasty expected exponential cost of low-Q environments.

We don't expect disputers to need to use the self-dispute-only structure, but this framework will encourage participation since it is always a viable fallback strategy that will preserve oracle accuracy in wild market conditions.

In general, having many disputers reduces costs for everybody and is an extremely powerful network effect.
