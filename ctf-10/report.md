# Sample Report Template

## Challenge 10: *Mistilteinn*

### Description

Whitelisted user can circumvent mint limit by transferring minted NFTs to another address.

The check in ./contract.rs#L95-107 only looks at the current snapshot of the sender's held NFTs. An attacker can simply mint, and then transfer the NFT to another address. Upon a subsequent attacker call to ExecuteMsg::Mint, their resulting owned NFT count from Cw721QueryMsg::Tokens will not have increased.

### Recommendation

We recommend storing a cumulative mint count per whitelisted user, and checking against that on each call to ExecuteMsg::Mint.

### Proof of concept

```rust

```
