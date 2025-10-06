Proof of stake systems are vulnerable to censorship bribery attacks, where a set of validators accepts payment for the successful exclusion of some set of transactions over some time period. Attempts to patch this critical vulnerability include social slashing and FOCIL, among others. At the end of the day; however, it is very difficult to prove censorship and even with perfect slashing, there is only 32 eth per block available to slash. It is certain there are payoffs far in excess of this tiny amount of ETH lost by the bribee. With recent advancements in zero knowledge technology as well as the proliferation of cheap blockspace, it is reasonable to assume private censorship bribery infrastructure will be common in just a few years' time if not sooner.

One can even imagine a simple payment-for-future-state smart contract, where a briber puts up say 100k eth, and specifies a contract state they want to not change, while also specifying any methods that could mutate said state. Validators sign a simple transaction recording one bribee address in each block, and since they are the block producer, they can ensure they are the first one to sign it. After the censorship period expires, someone attests to the non-existence of mutating transactions in the blockchain with a bond. There is then a challenge period, and if unchallenged, the attester receives say 10k eth and the remaining 90k is split among the bribee address list. If challenged, the challenger can prove a mutating function was called and included in the block period and claim the attester's bond.

There are far more sinister things an attacker can exploit in the proof of stake context, [as outlined by Zack from Amoveo](https://github.com/zack-bitcoin/amoveo-docs/blob/master/other_blockchains/proof_of_stake.md). But for now we will assume there is no cartel preventing new validators from joining the validation set, but still everybody is taking bribes for immediate censorship profit.

Clearly this is an issue if a 300 line smart contract can change the consensus metagame so easily / completely destroy most of DeFi in its current form. Social slashing is not a credible deterrent, and third party interventions on the blockchain to purge the censorship regime is not scaleable nor is it secure. Clearly Ethereum should switch to proof of work which does not have the same censorship bribery issues because of the propensity for defection by 51% censorship coalitions into subcoalitions. Said another way, censorship bribery is not a stable equilibrium in proof of work and it fixes itself on its own without third party intervention. But that will probably not happen until it is too late. Ethereum needing to switch to proof of work is not an opinion unless someone can come up with a proof of stake design stable against censorship bribery. Definitionally it is not possible, since in proof of stake systems influence over consensus is proportional to values internal to the system, strictly.

So we want to see what the impacts are of this kind of inevitable proof of stake censorship bribery regime on openOracle. If you can't get a dispute out, a bad price can be settled.

We will first frame exactly what adversarial blockchain context we are working in when we say "censorship bribery regime".

Censorship context:
Let's assume all block producers are accepting bribes, and if the bribe is big enough, they will not include a certain class of transactions, disputes here. Assume zero altruistic block producers. Assume perfect bribery infrastructure (all-or-nothing to all those taking the bribe) with 100% participation and zero slashing.

Note the zero slashing part can be expanded to 32 eth slashed per block of censorship or skipped block in FOCIL if desired and the economics don't change for large amounts.

Assume the attacker submits a bad price, and tries to censor all disputes with bribe payments for the settlementTime period.

Since the only way to pass price error from inside the oracle to the external notional is to offer a badly priced swap, a link between the initial oracle liquidity and external notional I = (initial liquidity / external notional) implies a 1 / I number of blocks of bribery payments until the possible extractable value is eaten by gas costs paid to the bribe acceptor, for linear external notionals. We can more generally say for linear external notional we can configure our oracle instance to a desired censorship attack grief ratio G by setting settlementTime in blocks to (G+1)/I.

We can prove this with a simple example. Imagine a $1m perp long, with a price settled by an openOracle initial report with $100k of initial liquidity in each token. If an attacker wanted to submit a price that was 5% off, they would need to pay >$5k per block since that is the dispute profit available for a block producer to include a dispute. If they can extract 5% on $1m, this is $50k of profit. So if the settlement time is > 10 blocks, the possible extraction from the perp notional is strictly less than the cost to bribe block producers to exclude disputes. 

Next, under FOCIL, we are assuming the inclusion committee members get routed fees. I can't imagine why this wouldn't happen but I guess it is possible so this next part is not bulletproof. But if we have some rotating committee of size M, for linear external notional we say:

  number of blocks with grief ratio G is (G + 1) / (I * M)

We have to look at binary notionals now. Assume no FOCIL. For example, let's say someone bets a large amount that the price is below some number N. They report a number just below N and then censor all disputes. The true price is at P. We can say, the internal oracle price error here is (P-N)/P, call that a delta "D". This delta must be paid for by the briber. So for a binary outcome:

  number of blocks with grief ratio G is (G + 1) / (D * I)

Next, we are making assumptions about external notional, but clearly more notional can latch on and act as "parasitic open interest," a common vulnerability in oracle designs.

Let's start from the (G+1) / I assumption here.

You could say, parasitic open interest drives up external notional, which decreases I and so increases the amount of blocks for a given grief ratio G in an unbounded way.

Let's describe what "parasitic open interest" would look like in openOracle. It is not a price feed. So you would need standing liquidity that always is willing to make some bet based on any reportId that fits within its parameters. Next, recall we are in a perfectly efficient censorship bribery blockchain context.

Imagine a situation where there's 10x as many blocks that are actually needed for a grief ratio of > 0. Parasitic OI can latch onto the reportId (if they know the external notional) and possibly be safe up to where the censorship bribery attack turns profitable. The problem for the latcher is, the existence of other standing liquidity means the attacker who is doing the censorship bribes can add their own OI to the reportId and then anyone who latched on blows up.

Standing liquidity that is "censorship-attack aware" immediately blows up because of this. It may take an oracle user with it which is unfortunate. But still the concept of "the standing liquidity / parasitic OI is dead well before YOU do a price request" is statistically true.

So you could say, i want to assign a grief ratio to the censorship briber G and then also a grief ratio to the parasitic OI latchers of P or something:

  (G+1) / I becomes (max(G,P) + 1) / I

It is left as an exercise to the reader to calculate the required number of blocks for a given grief ratio to prevent something like a Chainlink or Pyth price from getting on-chain in comparison to openOracle.
