There are various ways people have tried to solve the generalized oracle problem. One proposal that does not rely on ecosystem forking is SchellingCoin, [introduced by Vitalik in 2014](https://blog.ethereum.org/2014/03/28/schellingcoin-a-minimal-trust-universal-data-feed).

Very roughly, the game is as follows. There are reporters submitting sealed reports and they are rewarded P if they voted with the majority at reveal. The idea is the truth is a schelling point for reporters. Aside from the P + epsilon attack, there is an issue of deciding who is in the participation set.

From Vitalik's piece linked above, the fundamental mechanism is as follows:

"Suppose you and another prisoner are kept in separate rooms, and the guards give you two identical pieces of paper with a few numbers on them. If both of you choose the same number, then you will be released; otherwise, because human rights are not particularly relevant in the land of game theory, you will be thrown in solitary confinement for the rest of your lives. The numbers are as follows:

14237 59049 76241 81259 90215 100000 132156 157604

Which number do you pick?"

100000 is the obvious answer. We can try to create this game on the blockchain without voting. A naive approach: A reporter seals an answer, anyone can match them with a sealed answer of their own, if they agree after reveal, the answer stands, they split some reward, and all is good. If they disagree, they are both slashed. There is a trivial exploit with the reporter and matcher being the same entity and reporting nonsense. So we need open participation.

Instead of the naive prisoner game, imagine the following: You have a reporter reward. Anyone can report a sealed value by committing a fixed amount of liquidity. If after reveal everyone agrees, they split the reward. If anyone disagrees, all the reporters lose their money and some portion funds a larger next round. Continuation value can become nasty here so need to be mindful in an implementation.

In terms of P + Epsilon, where P is the reward and Q is the loss, the bribe is actually P + Q + E. We assume ground truth is 0.

(Your report, others' reports) = Your payoff

Payoffs, happy path:
- (1, any 0) = -Q
- (1, all 1) = +P
- (0, any 1) = -Q
- (0, all 0) = +P

P+Q+E bribery:
- (1, any 0) = +(P+E)
- (1, all 1) = +P
- (0, any 1) = -Q
- (0, all 0) = +P

From this simplified perspective, we still have a P + epsilon vulnerability. But we don't have the same kind of Sybil / participation set problem. It's a permissionless game.

One attack vector is spamming reports to discourage participation, since the reward is fixed and split across reporters. One approach is taking advantage of continuation value. If you make the next round's rewards larger when there are more reports, it can become worth it to report two opposing answers, or just the wrong answer, and move the game to the next round. So we are trying to turn the report spam against the attacker. 

The continuation value dynamic may interfere with the P + Q + E attack economics in the above simplified perspective, which would be a good thing (may make bribery more difficult). The briber still has to pay the 1 reporter if anyone reported 0, which they may do to capture continuation value. Also, bribery may increase reporter participation, which directly increases continuation value, since the next round is funded from the reporters' stakes.

It seems like continuation value is higher under manipulation than with honest reporting, which may allow us to avoid degenerate escalation when we don't want it. That would be a nice result. Another way of saying this: manipulation (bribery or spam) may endogenously create more continuation value than honest resolution, and that extra continuation value may be capturable by participants in a way that raises the effective cost of bribery.

One of the consequences of continuation value causing next rounds is the marginal reporter will report up to an aggregate liquidity across all reporters for that round somewhere under the continuation trigger. This means a manipulator can calculate themselves where this threshold is, then flood the zone with all lies up to the reporter indifference threshold, hoping to get a lie through. The problem is, the limited number of "safe" seats creates a race dynamic among the honest reporters. The manipulator has to occupy all the seats very quickly, because if a single other reporter reports ground truth, they just donated heavily to the next round. The speed of occupation then leaks information to continuation value farmers, who might find the next round more attractive, even if the aggregate liquidity is just under the honest continuation threshold, which punishes the manipulator. More simply, playing near the no-manipulation continuation value breakeven is a bold move when you leak information that increases continuation value. Granted it is very possible the mechanism fails here, and the flood-the-zone attack works.

It may not be possible to have full receipt-freeness onchain. There are also challenges with preventing selective non-reveal from becoming a strategic weapon. In order to preserve the lack of knowledge of the other's report in the classic prisoner game, we want a scheme where, during a live window, a participant cannot provide binding evidence of their true report, but after a delay the protocol can verify unanimity versus disagreement.

Overall, the game is extremely strange and might not work. Maybe we can't make it resistant to bribery. Maybe the continuation dynamic is too knife-edge between good reporting and exponential clown world blowups. Also it's certainly possible the schelling point has nothing to do with ground truth.

In the event we get the game working, its intended use is per-instance, where the requester parameterizes the game as they would like. If we can find a parameterization whose equilibrium is escalate under attack & don't escalate under honesty, then we have found a scale-invariant solution to the generalized oracle problem. We don't need a perfect general oracle, just one that isn't structurally broken at scale like nearly all the other designs. Speculatively, the solution was never going to be some nice object anyways. Deranged solutions for deranged problems.
