# Sample Report Template

## Challenge 07: *Tyrfing*

### Description

The top depositor (./contract.rs#L14) and owner (./state.rs#4) share the same storage key "address", leading to the top depositor having privilege to call ExecuteMsg::OwnerAction.

An attacker may call ExecuteMsg::Deposit with DENOM quantity greater than THRESHOLD to set their address as the top depositor/owner, and then call ExecuteMsg::OwnerAction with BankMsg::Send to drain funds.


### Recommendation

We recommend updating the storage keys so that TOP_DEPOSITOR and OWNER have different values.

```rust

// CURRENT
// in contract.rs#L14
pub const TOP_DEPOSITOR: Item<Addr> = Item::new("address");
...

// in state.rs#L4
pub const OWNER: Item<Addr> = Item::new("address");


// RECOMMENDED
// in contract.rs#L14
pub const TOP_DEPOSITOR: Item<Addr> = Item::new("top_depositor");
...

// in state.rs#L4
pub const OWNER: Item<Addr> = Item::new("owner");
```

### Proof of concept

exploit shown here: https://github.com/bear-market-labs/oak_ctf/commit/04ae0756e890a40d15d2260fde5d009b5814aeb5#diff-2b3053e3d2b0922f2e5a13d5ec8e7f08b0d0ccdc7b5f93023a0907b7c1676f11

```rust

```
