# Sample Report Template

## Challenge 04: *Gram*

### Description

A user can withdraw more than deposit via inflation attack (only for initial deposit).

Contracts that mint/burn shares commensurate with a proportional share of a pool or supply are susceptible to an inflation attack, where an attacker frontruns the contract's initial deposit with their own deposit plus a "donation" that manipulates a victim's minted shares.

Attack flow:

1. Share vault is deployed
2. Attacker deposits 1 unit for 1 share of the pool
3. Share vault's first non-attacker deposit of 100 submitted to mempool
4. Attacker frontruns with a donation of 50
5. The deposit from step #3 is processed, and receives 100 * (1 / (1 + 50)) = 1 share (due to rounding)
6. Attacker withdraws his 1 share for 75 units (50% of the vault), which is more than his 51 unit investment

### Recommendation

As hinted in msg.rs#L8, the recommendation is to designate some 10**offset amount of share tokens to be perma-locked in the vault upon the genesis deposit.


```rust
// in ./contract.rs#L58

// CURRENT
let mint_amount = if total_supply.is_zero() {
    amount
} else {
    amount.multiply_ratio(total_supply, total_assets)
};

// RECOMMENDED
let MIN_SUPPLY = Uint128::from(1000u32);
let mint_amount = if total_supply.is_zero() {
    
    if amount < MIN_SUPPLY{
      //return error
    }
    
    BALANCES.update(
        deps.storage,
        &env.contract.address,
        env.block.height,
        |balance| -> StdResult<_> { Ok(balance.unwrap_or_default() + MIN_SUPPLY) },
    )?;
    config.total_supply += MIN_SUPPLY;

    amount - MIN_SUPPLY
} else {
    amount.multiply_ratio(total_supply, total_assets)
};
```

### Proof of concept

exploit shown here: https://github.com/bear-market-labs/oak_ctf/commit/04ae0756e890a40d15d2260fde5d009b5814aeb5#diff-cb6ad46de45b02c32849c02ab01dc46a7d1821ea1775b58829660a489066e39f

```rust

```
