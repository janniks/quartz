---
tldraw: https://www.tldraw.com/r/Z_a7b6upQMQkqT4de8AJE?v=409,435,1280,1335&p=page
---
## Reward Cycles

This is how Stacks cycles are often displayed. So, naturally, many people think a Stacks cycle has its prepare phase at the front (also called the burn phase sometimes).
![[65bdbd79-b276-4b5a-8298-b6e432f484f3.png]]
Careful, that's only partially correct.


Here's a better mental model of a cycle.
The important difference is that prepare phases are actually at the end of a cycle—and they "prepare" the upcoming cycle.
![[e7c01fb5-6cc1-41da-9b89-e5cbd3ac1083.png]]
![[d3ba3399-e516-44a5-8ae5-e675d188682a.png]]
It turns out, this isn't the full picture either.


More precisely, the prepare-phase is sort of shifted to the right by one.
The first block of the prepare-phase is one later than you might expect, given a prepare phase length.
Also, the first (index `0`) block of a cycle is still technically in the prepare-phase and does not have rewards.
![[62cc41f4-791c-4f11-8196-bfdcd052f901.png]]

That means, to make your stacking txs count towards the next cycle, make sure they are submitted **before** the pox-anchor block (first block of the prepare phase).
The state from that block will be used for determining reward slots for the next cycle.
![[cdb9aa92-0f56-4f3a-8ecb-0cd65126b430.png]]

---
# General Notes
*Some facts upfront*
- /v2/pox `.current_burnchain_block_height` refers to the last mined BTC block height. The content of this block is "already on the blockchain"

*For simplicity we assume the following lengths in examples (taken from the regtest environment)
- reward cycle length = 20
- reward phase length = 15
- prepare phase length = 5
- so, an index can fall on modulo `% 20` i.e. index `0-19`
## Prepare phase
- The first block "in the prepare phase" determines the stacking-state used for the upcoming reward cycle. The block is called the pox-anchor-block and the stacks-block event includes the `reward_set` for the upcoming cycle.
## Off-by-one weirdness
- The prepare phase is shifted to the right by one and overlaps "into the next cycle" according to the core code.
	- Blocks index `1-15` are the reward phase
	- Blocks index `16-19` AND `0` are the prepare phase
	- Block index `16` determines the stacking state
		- If we stack at `current_burn_height` == `15` we can still get our stacking-txs into the next block (`16`)
- Funds are unlocked once we hit `unlock_height + 1`
	- If `current_burn_height` == `unlock_height` funds are still locked

## More pox-4 quirks
The `reward-cycle` argument for stacking functions refers to different cycles.
It should be:
- the current cycle for normal stacking
- the next/target cycle for delegation commits

# Method Notes
## `stack-stx`
- Meant for solo-stackers
- The signer-key can't be changed for the amount of cycles given
- `params`
	- `start-burn-ht` should be the current burn height
	- `signature`
		- `cycle` refers to current cycle
		- `period` refers to the lock period (in cycles)

## `stack-increase`
- Can "increase" the amount stacked for a stack
- Needs to use the same signer-key as the stack
- `params`
	- `signature`
		- `cycle` refers to current cycle
		- `period` needs to be the same as the original cycles

## `stack-extend`
- Can "extend" stacking without a cooldown
- This creates new entries, so we can use a new/different signer-key
- `params`
	- `signature`
		- `cycle` refers to current cycle
		- `period` is the extend-by period

## `stack-aggregation-commit-indexed`
- `params`
	- `signature`
		- `cycle` refers to target cycle

## `stack-aggregation-increase`
- `params`
	- `signature`
		- `cycle` refers to target cycle


