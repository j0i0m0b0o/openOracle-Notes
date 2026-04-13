There are various ways people have tried to solve the generalized oracle problem. One proposal that does not rely on ecosystem forking is SchellingCoin, [introduced by Vitalik in 2014](https://blog.ethereum.org/2014/03/28/schellingcoin-a-minimal-trust-universal-data-feed).

Very roughly, the game is as follows. There are reporters submitting sealed reports and they are rewarded P if they voted with the majority at reveal. The idea is the truth is a schelling point for reporters. Aside from the P + epsilon attack, there is an issue of deciding who is in the participation set.

From Vitalik's piece linked above, the fundamental mechanism is as follows:

"Suppose you and another prisoner are kept in separate rooms, and the guards give you two identical pieces of paper with a few numbers on them. If both of you choose the same number, then you will be released; otherwise, because human rights are not particularly relevant in the land of game theory, you will be thrown in solitary confinement for the rest of your lives. The numbers are as follows:

14237 59049 76241 81259 90215 100000 132156 157604

Which number do you pick?"

100000 is the obvious answer. We can try to create this game on the blockchain without voting. A naive approach: A reporter seals an answer, anyone can match them with a sealed answer of their own, if they agree after reveal, the answer stands, they split some reward, and all is good. If they disagree, they are both slashed. There is a trivial exploit with the reporter and matcher being the same entity and reporting nonsense. So we need open participation.

Instead of the naive prisoner game, imagine the following: You have a reporter reward. Anyone can report a sealed value by committing a fixed amount of liquidity. If after reveal everyone agrees (unanimity), they split the reward and this answer stands as the oracle output. If anyone disagrees, all the liquidity becomes the reward for the next round, which has the same fixed liquidity commitment to participate, and we play the game again. Naturally, we avoid needing to decide who is in the participant set and deal with Sybils in the strong sense by making identity irrelevant.

In order to preserve the prisoners' lack of knowledge of others' reports, we need a cryptographic scheme where, from start to unanimity, a participant cannot provide binding evidence of their true report, but after a delay the protocol can verify unanimity versus disagreement each round. Maybe this is not possible without introducing some trusted component. Unanimity versus disagreement is at least just an equality predicate.

In terms of P + Epsilon, where P is the reward and Q is the loss, the bribe is actually P + Q + E. We assume ground truth is 0. Assume P + Q + E is paid to all who report 1, but only when there is at least one 0 report.

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

Since it is always worth it to report 1, everybody does, the oracle resolves to 1 incorrectly, and the briber doesn't have to pay anything.

But the continuation value dynamic interferes with the P + Q + E attack economics, which is a good thing. The briber has to pay all 1 reporters if anyone reported 0, which they may do to capture continuation value. Also, bribery may increase reporter participation, which directly increases continuation value, since the next round is funded from the sum of current reporters' stakes. The briber can be taken advantage of by continuation value farmers, since their costs are now reduced. It is very hard and very expensive for the briber to bribe all reporters. There isn't a static, known participant set.

One of the consequences of continuation value causing next rounds is the marginal reporter may report up to an aggregate liquidity across all reporters for that round somewhere under the continuation trigger. This means a manipulator can calculate themselves where this threshold is, then flood the zone with all lies up to the reporter indifference threshold, hoping to get a lie through. The problem is, the limited number of "safe" seats creates a race dynamic among the honest reporters. The manipulator has to occupy all the seats very quickly, because if a single other reporter reports ground truth, they just donated heavily to the next round. The speed of occupation then leaks information to continuation value farmers, who might find the next round more attractive, even if the aggregate liquidity is just under the honest continuation threshold, which punishes the manipulator. More simply, playing near the no-manipulation continuation value breakeven is a bold move when you leak information that increases continuation value. Granted it is very possible the mechanism fails here, and the flood-the-zone attack works. It is also possible the concept of a "marginal honest reporter" is not well specified. Maybe everyone nuts enough to play this game is a continuation value farmer.

It seems like continuation value is higher under manipulation than with honest reporting, which may allow us to avoid degenerate escalation when we don't want it. That would be a nice result, but is not strictly needed, since if continuation just causes delay without the cost of false resolution it isn't a big deal. Another way of saying this: manipulation (bribery or spam) may endogenously create more continuation value than honest resolution, and that extra continuation value may be capturable by participants in a way that raises the effective cost of bribery.

Overall, the game is extremely strange, very aggressive, and might not work. On the bright side, the derivation falls straight out of trying to implement the prisoner game on-chain. It's certainly possible the schelling point has nothing to do with ground truth, but it seems like deep into a runaway continuation game, the most likely option to end the game and earn the very large reward is still ground truth.

This means there can be nasty continuation structure yet the game still finalizes on ground truth. In principle, continuation value from the reward growing each round can attract an extreme amount of capital by the time unanimity is reached, which may incentivize an ecosystem fork.

In the event we get the game working, its intended use is per-instance, where the requester parameterizes the game as they would like. If we can find a parameterization whose equilibrium is escalate under attack & don't escalate under honesty, then we have found a scale-invariant solution to the generalized oracle problem. We don't need a perfect general oracle, just one that isn't structurally broken at scale like nearly all the other designs. Speculatively, the solution was never going to be some nice object anyways. Deranged solutions for deranged problems.

----------------------------------------

Required crypto scheme:

  1. A reporter chooses a value x.
  2. During the live window, nobody else learns x.
  3. During the live window, the reporter also cannot produce a convincing receipt proving x to anyone.
  4. After a VDF delay seeded by future block entropy, anyone can produce a publicly verifiable proof of exactly one of:
     UNANIMOUS or DISAGREEMENT.
  5. Intermediate-round values remain hidden.
  6. On terminal unanimity, the unanimous value is revealed.
  7. No committee.
  8. No trusted dealer.
  9. No hardware assumption.

  The scheme is not available from standard primitives. In principle, this does not appear impossible, but likely requires very exotic cryptography.

  Deniability is not there to make bribery impossible. The point is to preserve the separate room condition of the prisoner game, so the Schelling point cannot form around what others are currently reporting.

