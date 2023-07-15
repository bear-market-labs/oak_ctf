# Sample Report Template

## Challenge 08: *Gjallarhorn*

### Description

NFT trade taker may revoke approval after calling ExecuteMsg::NewTrade to prevent ExecuteMsg::AcceptTrade from sending the trade taker's NFT.

In ./contract.rs#L257-270, ExecuteMsg::AcceptTrade dispatches the two NFT transfer submessages to execute the trade. The issue is both submessages' reply is set to "reply_always", which will execute logic that will always return an Ok() result (and persist state) even if the submessage results in an error.

An attacker may call ExecuteMsg::NewTrade, and then revoke the approval. If the seller calls ExecuteMsg::AcceptTrade, the contract will encounter an error when attempting to transfer the attacker's NFT, but the reply logic is will always return an Ok() result.

### Recommendation

We recommend only executing the reply logic on success so that errors revert the transaction and do not change the state.

```rust

// CURRENT
let mut submsgs = vec![SubMsg::reply_always(
  ...
)];

// Offered
submsgs.push(SubMsg::reply_always(
...
));

// RECOMMENDED
let mut submsgs = vec![SubMsg::reply_on_success(
  ...
)];

// Offered
submsgs.push(SubMsg::reply_on_success(
...
));
```

### Proof of concept

```rust

```
