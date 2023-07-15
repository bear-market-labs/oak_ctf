# Sample Report Template

## Challenge 02: *Gungnir*

### Description

User can vote multiple times for proposal lengths longer than LOCK_PERIOD.

The normal full-cycle flow for a voter is:

1. ExecuteMsg::Deposit
2. ExecuteMsg::Stake
3. GovExecuteMsg::Vote
4. ExecuteMsg::Unstake (after waiting for LOCK_PERIOD seconds)
5. ExecuteMsg::Withdraw

The issue is if a proposal's active length is greater than LOCK_PERIOD, an attacker can sequentially vote multiple times:

6. Send withdrawn funds to different address
7. Go to #1 if the proposal is still active

  
### Recommendation

We recommend using the cw_storage_plus::SnapshotMap data structure for VOTING_POWER in state.rs:

- Store a proposal's creation block upon submission (in external gov-like contract)
- Update QueryMsg::GetUser and QueryMsg::GetVotingPower to accept an argument such as height: u64
- Reference the updated QueryMsg::GetVotingPower (or GetUser) using the proposal's creation block when casting votes in external gov contract

### Proof of concept

```rust
// not confident of this answer, and did not write a POC exploit
```
