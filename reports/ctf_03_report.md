# Sample Report Template

## Challenge 03: *Laevateinn*

### Description

Unprivileged user can set the flash loan contract's owner to any address.

This unconventional flash loan setup separates the flash loan balance custodian and public entry point into two contracts, flash_loan and proxy respectively, with mutual trust conditions and privileges. In proxy/contract.rs#L59, the check to prevent calling the flash_loan contract does not account for casing differences.

An attacker can call the proxy contract's ExecuteMsg::RequestFlashLoan message with recipient set to an uppercase version of the flash loan contract, and msg set to the flash loan contract's ExecuteMsg::TransferOwner message (with their choice of new_owner). 
  
### Recommendation

We recommend two changes:

1. Update the check in proxy/contract.rs#L59 to cover casing differences

```rust
// in proxy/contract.rs#L59

// CURRENT
if recipient == config.flash_loan_addr{
  ...
}

// RECOMMENDED
if deps.api.addr_validate(recipient.as_str())? == config.flash_loan_addr {
  ...
}
```

Also, alternatively, the RequestFlashLoan message can remove the recipient parameter, and always send funds to info.sender.

2. Do not give the proxy contract call privileges to the flash loan contract's ExecuteMsg::TransferOwner

```rust
// in flash_loan/contract.rs#L184

// CURRENT
if !is_trusted(&info.sender, &config) { 
    ...
}

// RECOMMENDED
if info.sender != config.owner{
  ...
}

```

### Proof of concept

exploit shown here: https://github.com/bear-market-labs/oak_ctf/commit/04ae0756e890a40d15d2260fde5d009b5814aeb5#diff-9cbb2103013938b05c05044cbe98501a3caebf89b6de7119da1ef265c1942a39

```rust

```
