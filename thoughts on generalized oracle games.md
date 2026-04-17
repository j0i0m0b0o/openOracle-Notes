
## SchellingCoin
There are various ways people have tried to solve the generalized oracle problem. One proposal that does not rely on ecosystem forking is SchellingCoin, [introduced by Vitalik in 2014](https://blog.ethereum.org/2014/03/28/schellingcoin-a-minimal-trust-universal-data-feed).

Very roughly, the game is as follows. There are reporters submitting sealed reports and they are rewarded P if they voted with the majority at reveal. The idea is the truth is a schelling point for reporters. Aside from the P + epsilon attack, there is an issue of deciding who is in the participation set.

From Vitalik's piece linked above, the fundamental mechanism is as follows:

"Suppose you and another prisoner are kept in separate rooms, and the guards give you two identical pieces of paper with a few numbers on them. If both of you choose the same number, then you will be released; otherwise, because human rights are not particularly relevant in the land of game theory, you will be thrown in solitary confinement for the rest of your lives. The numbers are as follows:

14237 59049 76241 81259 90215 100000 132156 157604

Which number do you pick?"

100000 is the obvious answer. We can try to create this game on the blockchain without voting. A naive approach: A reporter seals an answer, anyone can match them with a sealed answer of their own, if they agree after reveal, the answer stands, they split some reward, and all is good. If they disagree, they are both slashed. There is a trivial exploit with the reporter and matcher being the same entity and reporting nonsense. So we need open participation.

## Moving the prisoner game on-chain
Instead of the naive prisoner game, imagine the following: You have a reporter reward. Anyone can report a sealed value by committing a fixed amount of liquidity. If the sealed reports agree (unanimity), reporters split the reward and this answer stands as the oracle output. If anyone disagrees, all the liquidity adds to the reward for the next round, with the same fixed liquidity commitment to participate, and we play the game again with the same rules. Naturally, we avoid needing to decide who is in the participant set and deal with Sybils in the strong sense by making identity irrelevant.

In order to preserve the prisoners' lack of knowledge of others' reports, we need a cryptographic scheme where a participant cannot provide binding evidence of their true report, but after a delay the protocol can verify unanimity versus disagreement each round. Maybe this is not possible without introducing some trusted component. Unanimity versus disagreement is at least just an equality predicate.

## Bribery
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

But the continuation value dynamic interferes with the P + Q + E attack economics. The briber has to pay all 1 reporters if anyone reported 0, which they may do to capture continuation value. Also, bribery may increase reporter participation, which directly increases continuation value, since the next round is funded from the sum of current reporters' stakes. The briber can be taken advantage of by continuation value farmers, since their costs are now reduced. It is very hard and very expensive for the briber to bribe all reporters. There isn't a static, known participant set.

## Flood the zone
One of the consequences of continuation value causing next rounds is the marginal reporter may report up to an aggregate liquidity across all reporters for that round somewhere under the continuation trigger. This means a manipulator can calculate themselves where this threshold is, then flood the zone with all lies up to the reporter indifference threshold, hoping to get a lie through. The problem is, the limited number of "safe" seats creates a race dynamic among the honest reporters. The manipulator has to occupy all the seats very quickly, because if a single other reporter reports ground truth, they just donated heavily to the next round. The speed of occupation then leaks information to continuation value farmers, who might find the next round more attractive, even if the aggregate liquidity is just under the honest continuation threshold, which punishes the manipulator. More simply, playing near the no-manipulation continuation value breakeven is a bold move when you leak information that increases continuation value. Granted it is very possible the mechanism fails here, and the flood-the-zone attack works. It is also possible the concept of a "marginal honest reporter" is not well specified. Maybe everyone nuts enough to play this game is a continuation value farmer.

It seems like continuation value is higher under manipulation than with honest reporting, which may allow us to avoid degenerate escalation when we don't want it. That would be a nice result, but is not strictly needed, since if continuation just causes delay without the cost of false resolution it isn't a big deal. Another way of saying this: manipulation (bribery or spam) may endogenously create more continuation value than honest resolution, and that extra continuation value may be capturable by participants in a way that raises the effective cost of bribery and spam.

## Runaway escalation
Deep into a continuation game, there are likely many participants in each new round. The pot has grown substantially while seat cost stays fixed. With large participation and real deniability, the terminal condition becomes a Schelling problem over a large anonymous crowd. That seems more naturally favorable to truth than to any specific lie. So, absent a concrete coordination story for the lie, in my view the burden is on the skeptic to explain why false terminal unanimity here should be easy or likely. 

"Cheap talk can degrade participation and finalize a lie even in extreme continuation" is one argument, but it fights a tough EV battle against a permissionless, deniable, unknown set. For a given total reward, the fewer the participants, the more valuable each seat is. Since we've had many rounds, the total reward is very high. Nobody can prove what is actually going on inside the game beyond their own reporting. Also, when the pot has grown enough, the value in the game can dwarf the value depending on the oracle output the liar wants finalized in their favor, which calls into question the strategic objective of getting a lie to finalize. The manipulator has to make oracle game moves anyways, where optimal control may say to play it in a different way. For example, the person committing to keep extending the game to get a lie to finalize might just want to thin participation: As participants thin, they may still be reporting ground truth. The manipulator can now report alongside them and earn a large payout. The mechanics are strongly influenced by the size of the external notional relative to the oracle game.

Finally, continuation value scales proportionally to the credibility of cheap talk (may involve publishing signed messages by large wallet, or reputation) truly dedicated to the lie side. It may become even more worth it to report on ground truth, since if the round finalizes, you split the pot, and if not, the game moved into a more valuable state. If you report on the lie, you decrease the chance of continuation.  In that moment continuation over not just the next round, but also rounds after that, is proportional to credibility. The decision to credibly commit to a lie still depends on external notional versus internal oracle liquidity at each decision point, since the manipulator does have to fire in at least the full liquidity each round. If they want to later try to recapture this, they are playing an unpleasant continuation game. Optimal control may say to capture a large part and risk a lot of money. They can try to capture a smaller part by risking less, but then they get less back.

The oracle game may actually be more reliable deep into continuation than at the beginning, since permissionless coordination over unanimity gets harder. The bet is when coordination gets that hard, the only report with a real chance of surviving is the strongest Schelling point. In principle, the mechanism may attract an extreme amount of capital by the time unanimity is reached, which may incentivize an ecosystem fork. With deniability, not much can be done aside from giving everyone their money back.

One way to reason about the mechanism is to put yourself in the shoes of someone on the sidelines deep into a continuation game. There is a massive reward for unanimity dwarfing the fixed seat cost. You have no credible proof of what anyone is doing in the game aside from disagreement in prior rounds. Do you participate? If you participate, which value do you report?

If delay seems like an Achilles' heel, the reporter liquidity requirement can be made to grow each round by a smaller % than the reward pot grows, instead of being fixed, capping out at some number. Long delay is equivalent to playing a near-exponential continuation game.

## Early game
Given our assumptions, the right place to focus is the early game if we want to break the oracle. If people want continuation, assuming there is not some seemingly credible commitment to lie already, they can do one of two things. They can just report a lie, or they can report a lie + truth. If there is seemingly credible commitment to a lie they can just report truth. Assuming there isn't a credible reporter, reporting a lie may seem cheaper but does not guarantee continuation. If everyone is doing this, the oracle can just settle on a lie, which is not ideal. On the other hand, the more continuation there is, the more valuable the later rounds are, which may dwarf the cost of the extra report. The other attack early on is flooding the zone, which we discussed above. It seems like if all participants not linked to the oracle output are trying to farm continuation value, the oracle gets stronger, since we leave the early game. Maybe the reward to seed the game early on in deployment needs to be higher to get enough participation. But in order to even be willing to report, you have to be willing to play in the next round to get your money back, or have a reward proportional to continuation risk.

A weird possibility is that a participant may rationally add a report, or even two disagreeing reports, simply because doing so directly increases continuation value. It may not take much to spark a kind of continuation value “singularity.” Also, if many people, or a large amount of capital, depend on the oracle output, expectations of future game value may revise sharply upward. Once we have the spark, we can get to the late game, where it seems brutally difficult to get a lie to finalize.

The game is terrible in the attrition sense. It isn't worth it to start playing, but then once you start playing, it isn't worth it to stop, and the amounts can be catastrophic. It is unfortunately structurally forced on the mechanism when you try to port the prisoner coordination game on-chain in a robust way. It is possible nobody wants to play.

## In conclusion
Overall, the game is extremely strange, very aggressive, and might not work. On the bright side, it falls straight out of trying to implement the prisoner game on-chain. It's certainly possible the schelling point has nothing to do with ground truth. Delay may be an issue for time-critical applications, however costly for the manipulator, but it does appear to strengthen resolution.

The game should not be thought of as a battle between honest and dishonest. There are really three strategic elements: continuation farming, finalization farming, and biasing the oracle output to influence some external dependent payout. The game needs to end eventually to capture the value. Truth may just incidentally be the cheapest terminal liquidation convention, or in simpler words the easiest way to end the game.

In the event we get it working, its intended use is per-instance, where the requester parameterizes the game as they would like. We don't need a perfect general oracle, just one that isn't structurally broken at scale like nearly all the other designs. Speculatively, the solution was never going to be some nice object anyways. Deranged solutions for deranged problems.

----------------------------------------

Required crypto scheme:

  1. A reporter chooses a value x.
  2. During the live window, nobody else learns x.
  3. Any receipt is deniable: a reporter who submitted x can produce an equally convincing receipt for any alternative value y.
  4. After a VDF delay seeded by future block entropy, anyone can produce a publicly verifiable proof of exactly one of:
     UNANIMOUS or DISAGREEMENT.
  5. Intermediate-round values remain hidden and deniable, except for information logically implied by the aggregate result together with a participant's own report.
  6. On terminal unanimity, the unanimous value is revealed.
  7. No committee.
  8. No trusted dealer.
  9. No hardware assumption.

  The full scheme is not available from standard primitives. In principle, this does not appear impossible, but likely requires very exotic cryptography.

  Deniability is not there to make bribery impossible. The point is to preserve the separate room condition of the prisoner game, so the Schelling point cannot form around credible evidence of what others are currently reporting.

  The clean primitive appears plausibly realizable from sufficiently strong iO style machinery: deniable report encodings for the input side, plus an obfuscated evaluator that uses the delayed VDF witness to reveal only the leakage function (unanimity versus disagreement), without exposing a general decryption capability that reveals individual reports. It may be quite some time before this is practical.

