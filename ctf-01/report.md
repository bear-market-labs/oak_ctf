# Sample Report Template

## Challenge 01: *Mjolnir*

### Description

Unprivileged user can drain all funds in the contract using duplicate ID values in a ExecuteMsg::Withdraw call.

In contract.rs, the withdraw function populates a Vec called lockups by looping through the ids parameter, and fetching the corresponding Lockup struct from storage (L82-84). However, if ids contain duplicate values, the withdraw function will still accumulate total_amount, which is sent to the caller.

An attacker can deposit to create a valid lockup, note the lockup ID (call it X), wait for the LOCK_PERIOD, and call ExecuteMsg::Withdraw with the ids field set as a vector with multiple duplicate X elements - ExecuteMsg::Withdraw{ ids: vec![X, X, X, X, X] }.
  
### Recommendation

We recommend deduping the ids Vec before contract.rs#L82.

```rust

let mut deduped_ids: Vec<u64> = ids.clone();
deduped_ids.sort();
deduped_ids.dedup();

// fetch vaults to process
for lockup_id in deduped_ids.clone() {
  ...
}

```


### Proof of concept

```rust


```
