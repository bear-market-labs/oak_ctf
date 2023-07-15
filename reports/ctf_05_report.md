# Sample Report Template

## Challenge 05: *Draupnir*

### Description

Any user can take ownership of vault through ExecuteMsg::AcceptOwnership.

In the two-stage ownership handoff process, the current owner designates a proposed owner in ExecuteMsg::ProposeNewOwner, and then the proposed owner calls ExecuteMsg::AcceptOwnership.

In ./contract.rs#L129-131, the check for the caller to match config.proposed_owner does not halt code execution when the if-condition is true. An attacker may compose a transaction to call ExecuteMsg::AcceptOwnership with any address, and then call ExecuteMsg::OwnerAction with a bank send message.

### Recommendation

We recommend updating ./contract.rs#L130 to return the ContractError.

```rust
// in ./contract.rs#129-131

// CURRENT
if state.proposed_owner != Some(info.sender.clone()) {
    ContractError::Unauthorized {};
}

// RECOMMENDED
if state.proposed_owner != Some(info.sender.clone()) {
    return Err(ContractError::Unauthorized {});
}
```

### Proof of concept

exploit shown here: https://github.com/bear-market-labs/oak_ctf/commit/04ae0756e890a40d15d2260fde5d009b5814aeb5#diff-495b0eeef951cfb8781c7495b8050630f7b4e713f064a8af56cb0e6d9ccf527e

```rust

```
