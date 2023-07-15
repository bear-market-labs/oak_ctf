# Sample Report Template

## Challenge 09: *Brisingamen*

### Description

Returning depositor who had previously fully withdrawn can claim disproportionate share of rewards.

In ./contract.rs#L180-182, there is a check to return out of the function if the user does not have any staked balance in the contract, skipping the critical update to a user's index in the case of re-depositing into the contract after having fully withdrawn. A user's index must always synchronize with the global index on any user action.

The attack flow:

1. Attacker deposits any number of tokens via address A.
2. Attacker deposits any number of tokens via address B, and then withdraws entire balance.
  a. after the block, the attacker's user_index is set to global_index=X
3. Owner distributes rewards and updates global_index to X+Y, with address A ensuring total_stake > 0
4. Attacker deposits an ideally large number of tokens via address B
  b. after the deposit, the attacker's user_index is *still* set to X
5. Attacker calls ExecuteMsg::Claim via address B, earning rewards from step 3

### Recommendation

We recommend removing lines ./contract.rs#L180-182 to ensure the user's reward information is always up to date.

### Proof of concept

exploit shown here: https://github.com/bear-market-labs/oak_ctf/commit/04ae0756e890a40d15d2260fde5d009b5814aeb5#diff-e6dfbb5c41551f6f0f0ad6366eff0f3782b921f3098dd29c16f845367f613278

```rust

```