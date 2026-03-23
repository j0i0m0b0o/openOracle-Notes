A useful primitive for cryptoeconomics would be a game that resolves into a purchasing power signal.

We propose the following game:

Whoever wants the signal offers a reward denominated in ETH.

To compete for the reward, a reporter stakes ETH and commits to a seed and a proof of work difficulty threshold. While the stake is live, anyone can try to break that reporter by producing a proof of work using the reporter’s seed and a nonce that clears the stated threshold. If such a proof is found, the reporter loses their staked ETH to the PoW provider. If not, they survive.

At the end of the game, the reporter with the most aggressive surviving difficulty threshold (most risky for the reporter) wins the reward.

Comparing the winning threshold across sequential games gives a signal about how ETH’s purchasing power changed over the interval, measured against computational cost. If the winning threshold rises from one game to the next, that suggests market participants were willing to burn more real world compute to win the staked ETH, implying stronger ETH purchasing power versus compute. If it falls, that suggests weaker purchasing power.

This does not measure purchasing power in a universal sense. Computational cost is at least anchored to physical reality, which may make it more stable than measuring against another token.

There are open questions, including but not limited to:

- How to consume the signal? If it's only binary, large changes in purchasing power have the same impact as small ones. If it's not only binary, manipulation and supply shocks have more effect
- How to mitigate the impact of faking purchasing power increases? It is easier to fake an increase in purchasing power than a decrease, since decreases require all miners to refuse to take some ETH reward. On the other hand, if this mechanism were used in a redemption rate controller, a purchasing power increase signal decreases the amount of backing asset a redeemer can earn, which seems to limit solvency damage
- How do censorship / block producer games factor in?
- Is it possible to introduce an openOracle-style forced succession mechanic into the game? For example, let's say the game starts with a 1 ETH reward, and reporters must stake 100 ETH. If a PoW solution is found that wins 100 ETH, 10 of that ETH seeds the reward for another game immediately, and reporters there must stake 1000 ETH, and so on. This way, faking purchasing power increases becomes exponentially expensive. Escalation multipliers are just for illustration.
- Does griefing completely kill honest tight reporting incentives / result in an equilibrium honest threshold that is manipulable? Is there a better way to do the forced succession in this context?

Overall, the purpose of this mechanism is not a perfect CPI oracle. We just want at least some tendril of reality, however messy, around which we can engineer.

-------------------

An orthogonal approach to this problem is to tokenize the proof of work and use it in a price ratio game. If you do this naively, the token has a ceiling price (miners can create more and sell if it's trading too high) but no floor, which makes using it to divine purchasing-power-adjacent things difficult. But, you could make the tokenized form redeemable for consensus weight, which gives it a price floor. This way, you could use it in an escalating two-way limit order game similar to openOracle. In PoW, proof of work is already redeemable for consensus weight, so it's just adding more to the standard process.

There may be very strange reflexive issues. It might not work. But it is at least possibly coherent.

One of the big challenges with this system is the token can't have consensus weight for too long, or you can accumulate and do deep reorgs. So that needs to be engineered around, if at all possible.
