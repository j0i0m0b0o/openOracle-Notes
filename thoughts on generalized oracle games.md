There are various ways people have tried to solve the generalized oracle problem. One proposal that does not rely on ecosystem forking is SchellingCoin, [introduced by Vitalik in 2014](https://blog.ethereum.org/2014/03/28/schellingcoin-a-minimal-trust-universal-data-feed).

Very roughly, the game is as follows. There are reporters submitting sealed reports and they are rewarded P if they voted with the majority at reveal. The idea is the truth is a schelling point for reporters. Aside from the P + epsilon attack, there is an issue of deciding who is in the participation set.

One way we can think about attacking the participant set problem is as follows. You have a reporter reward. Anyone can report a sealed value by committing a fixed amount of liquidity. If after reveal everyone agrees, they split the reward. If anyone disagrees, all the reporters lose their money and some portion funds a larger next round. Continuation value can become nasty here so need to be mindful in an implementation.

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

So we still have a P + epsilon vulnerability. But at least we don't have the same kind of Sybil / participation set problem. It's a permissionless game.

Maybe it isn't possible to have receipt-freeness onchain. There are also challenges with preventing selective non-reveal from becoming a strategic weapon. Maybe there is some cryptographic technique allowing full participant set reveal after some period of time, like with VDFs.

One major problem that needs to be solved is spamming reports to discourage participation, since the reward is fixed and split across reporters. One approach is taking advantage of continuation value. If you make the next round's rewards larger when there are more reports, it can become worth it to report two opposing answers, or just the wrong answer, and move the game to the next round. So we are trying to turn the report spam against the attacker. 

It is also possible the continuation value dynamic interferes with the P + Q + E attack economics, which would be a good thing (may make bribery more difficult). The briber still has to pay the 1 reporter if anyone reported 0, which they may do to capture continuation value. Also, bribery may increase reporter participation, which directly increases continuation value, since the next round is funded from the reporters' stakes.

It seems like continuation value is higher under manipulation than with honest reporting, which may allow us to avoid degenerate escalation when we don't want it.

Overall, the game is extremely strange and might not work. Maybe we can't make it resistant to bribery. Maybe the continuation dynamic is too knife-edge between good reporting and exponential clown world blowups. Also it's certainly possible the schelling point has nothing to do with ground truth.
