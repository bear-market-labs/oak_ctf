# Sample Report Template

## Challenge 06: *Hofund*

### Description

User can flash borrow voting tokens to take owner role, and repay loan via ExecuteMsg::OwnerAction.

In ./contract.rs#L127-145, ExecuteMsg::ResolveProposal compares the contract's balance of the voting token against the 1/3 supply threshold. The issue is cw20 tokens can be directly transferred to the contract via Cw20ExecuteMsg::Transfer, and avoid the checks in Cw20HookMsg::CastVote. 

The attack flow would be:

1. Attacker calls ExecuteMsg::Propose
2. Wait for voting window to pass
3. Take out a sufficient flash loan of the voting token; the msg payload is
  a. Call Cw20ExecuteMsg::Transfer to directly send the token into the contract and increase its balance
  b. Call ExecuteMsg::ResolveProposal, which will pass and give privileges to ExecuteMsg::OwnerAction
  c. Repay flash loan with ExecuteMsg::OwnerAction (execute another Cw20ExecuteMsg::Transfer to send contract's voting token holdings)

### Recommendation

We recommend adding a field in Proposal for holding cumulative votes, and only increasing it in Cw20HookMsg::CastVote, after the checks.

And then, this new field should be used in-lieu of the balance response in ./contract.rs#L136.

```rust

// CURRENT
Ok(Cw20HookMsg::CastVote {}) => {
    if config.voting_token != info.sender {
        return Err(ContractError::Unauthorized {});
    }

    if current_proposal 
        .timestamp
        .plus_seconds(config.voting_window)
        < env.block.time
    {
        return Err(ContractError::VotingWindowClosed {});
    }

    Ok(Response::default()
        .add_attribute("action", "Vote casting")
        .add_attribute("voter", cw20_msg.sender)
        .add_attribute("power", cw20_msg.amount))
}

// RECOMMENDED
Ok(Cw20HookMsg::CastVote {}) => {
    if config.voting_token != info.sender {
        return Err(ContractError::Unauthorized {});
    }

    if current_proposal 
        .timestamp
        .plus_seconds(config.voting_window)
        < env.block.time
    {
        return Err(ContractError::VotingWindowClosed {});
    }

    current_proposal.total_votes += cw20_msg.amount;
    PROPOSAL.save(deps.storage, &current_proposal)?;

    Ok(Response::default()
        .add_attribute("action", "Vote casting")
        .add_attribute("voter", cw20_msg.sender)
        .add_attribute("power", cw20_msg.amount))
}

```

### Proof of concept

exploit shown here: https://github.com/bear-market-labs/oak_ctf/commit/04ae0756e890a40d15d2260fde5d009b5814aeb5#diff-01080a685da9ebd8587154a47291b579566fc5c1c790e07d2214854de3be8a2f

```rust

```