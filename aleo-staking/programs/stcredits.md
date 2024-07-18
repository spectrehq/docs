- [Source code](https://github.com/spectrehq/spectre/tree/main/packages/leo/spectre_v1/stcredits)
- Deployed program (coming soon)

stcredits is the core program which acts as a liquid staking pool. The program is responsible for deposits, withdrawals, minting and burning liquid tokens, delegating funds to validators, applying fees and distributing rewards.

stcredits program also defines stCredits, an ARC-20 token that represents the account's share of the total supply of Credits tokens inside AleoStaking on Aleo system. It is a non-rebasable token, which means that the amount of tokens in the user's account is not going to change. During time, the value of this token is changing, since the amount of Credits tokens inside the protocol is not constant. stCredits will be integrated in variety of ZeFi applications across Aleo ecosystem.

## Mappings, Structs and Records

All the mappings can be read publicly, either by the user or by the programs.

#### total_supply

Returns the total supply of stCredits tokens.
The only valid key is `0u8`.

```Leo
mapping total_supply: u8 => u64;
```

#### account

Returns the public amount of stCredits tokens that the account owns.

```Leo
mapping account: address => u64;
```

#### approvals

Returns the amount of stCredits tokens that the spender is allowed to spend on behalf of the approver.
The key is the BHP256 hash of the `approval` struct instance.

```Leo
struct approval {
    approver: address,
    spender: address,
}

mapping approvals: field => u64;
```

#### config

Returns the global configuration of the liquid staking pool.
The only valid key is `0u8`.

```Leo
struct Config {
    paused: bool,
    treasury: address,
    protocol_fee: u8,
}

mapping config: u8 => Config;
```

#### state

Returns the state of the liquid staking pool.
The only valid key is `0u8`.

```Leo
struct State {
    withdraw: u64, // total withdrawal in credits
    pending_withdraw: u64, // total pending withdrawal in credits
    bonded: u64, // total bonded credits (it may have some lag)
    unbonding: u64, // total unbonding credits (it may have some lag)
    resolved_height: u32, // block height of the last resolved
}

mapping state: u8 => State;
```

#### cache_state

Returns the cache state of the liquid staking pool.
The only valid key is `0u8`.

```Leo
struct CacheState {
    status: u8,
    height: u32, // the start height or finished height
    bonded: u64,
    unbonding: u64,
    next_index: u32, // the index of the next delegator for caching
}

mapping cache_state: u8 => CacheState;
```

#### withdraws

Returns the withdrawal of a user account.

```Leo
struct Withdraw {
    amount: u64, // credits
    pending: bool, // whether the withdraw is pending
    height: u32, // block height at which the withdraw can be claimed or the pending withdraw **may** be claimed
}

mapping withdraws: address => Withdraw; // user => withdraw
```

#### delegators_count

Returns the number of delegators of the liquid staking pool.
The only valid key is `0u8`.
By this number, we can index the `delegators` mapping like an array.

```Leo
mapping delegators_count: u8 => u32;
```

#### delegators

Returns the delegator information, indexed by the delegator's index.
`delegators` and `delegators_count` enable us to iterate over all delegators of the liquid staking pool like an array.
The index satisfies: `0 <= index < delegators_count[0u8]`.

```Leo
struct Delegator {
    delegator: address,
    validator: address,
    bonded: u64,
}

mapping delegators: u32 => Delegator;
```

#### delegator_pos

Returns the position (index) of the delegator in the delegators array.

```Leo
mapping delegator_pos: address => u32;
```

#### validator_pos

Returns the position (index) of the validator in the delegators array.

```Leo
mapping validator_pos: address => u32;
```

## Transitions

#### get_metadata

Returns the metadata of the stCredits token.

```Leo
struct metadata {
    name: u128, // 16 bytes -> 16 characters with ASCII encoding
    symbol: u64, // 8 bytes -> 8 characters with ASCII encoding
    decimals: u8,
}
    
transition get_metadata() -> metadata;
```

#### approve_public

Approves the spender to spend the public amount of stCredits tokens on behalf of the caller.

```Leo
async transition approve_public(public spender: address, public amount: u64) -> Future;
```

#### unapprove_public

Unapproves the spender to spend the public amount of stCredits tokens on behalf of the caller.

```Leo
async transition unapprove_public(public spender: address, public amount: u64) -> Future;
```

#### transfer_from_public

Transfers the public amount of stCredits tokens from the approver to the receiver, signed by the spender.

```Leo
async transition transfer_from_public(public approver: address, public receiver: address, public amount: u64) -> Future ;
```

#### transfer_public

Transfers the public amount of stCredits tokens from the caller to the receiver.

```Leo
async transition transfer_public(public receiver: address, public amount: u64) -> Future;
```

#### transfer_private

Transfers the private amount of stCredits tokens from the sender to the receiver.
The inputs `sender`, `receiver` and `amount` are all private.
Returns the new private token records of the sender and the receiver.

```Leo
record token {
    owner: address,
    amount: u64,
}
    
transition transfer_private(sender: token, receiver: address, amount: u64) -> (token, token);
```

#### transfer_private_to_public

Transfers the amount of stCredits tokens from the sender to the receiver.
The token record of the sender is private.
Returns the new private token record of the sender.

```Leo
async transition transfer_private_to_public(sender: token, public receiver: address, public amount: u64) -> (token, Future);
```

#### transfer_public_to_private

Transfers the amount of stCredits tokens from the sender to the receiver.
Returns the new private token record of the receiver.

```Leo
async transition transfer_public_to_private(public receiver: address, public amount: u64) -> (token, Future);
```

### claim_unbond

Claim the amount of unbonded Credits from the validator of the delegator.
It will fail if the unbonding does not exist or the unbonding height is not reached.

```Leo
async transition claim_unbond(delegator: address) -> Future;
```

#### resolve_withdrawal

Resolve the total pending withdrawals of the liquid staking pool.
It is usually called after one or more transitions of `claim_unbond` has been executed.

```Leo
async transition resolve_withdrawal() -> Future;
```

#### supply

Supply (or deposit) the amount of Credits tokens to the liquid staking pool, and mint stCredits tokens to the caller.

```Leo
async transition supply(public credits: u64) -> Future;
```

#### withdraw

Withdraw the amount of Credits tokens from the liquid staking pool, and burn stCredits tokens from the caller.
It will create a withdrawal state or a pending withdrawal state for the user.

```Leo
async transition withdraw(public stcredits: u64) -> Future;
```

#### claim

Claim the amount of Credits tokens from the liquid staking pool.
There must be a withdrawal state or a pending withdrawal state for the user.
And the state must reach a condition which allows the user to claim the tokens.

```Leo
async transition claim(public amount: u64) -> Future;
```

## Operator Transitions

The following transitions can only be called by the operator.
The operator account should be controlled by the operator daemon service, which is maintained by the Spectre team and is managed by the admin (or the Spectre Governance).

#### bond

Can only be called by the delegator program's `bond` transition, to bond the amount of Credits tokens to the validator.
The signer must be the operator.

```Leo
async transition bond(public delegator: address, public validator: address, public amount: u64) -> Future;
```

#### unbond

Unbond the amount of Credits tokens from the validator of the delegator.
It will serve either for the users to withdraw Credits or for rebalancing bonding between the validators.

```Leo
async transition unbond(delegator: address, public amount: u64) -> Future;
```

#### resolve_withdrawal_force

Resolve the total pending withdrawals of the liquid staking pool.
It is usually called after one or more transitions of `claim_unbond` has been executed.
The difference between `resolve_withdrawal` and `resolve_withdrawal_force` is that the latter will force the resolution of the pending withdrawals to some extent, regardless of the liquidity of the pooled Credits amount in the liquid staking pool.

```Leo
async transition resolve_withdrawal_force(public credits: u64, public height: u32) -> Future;
```

#### cache

Will be called one or more times to cache the total bonded/unbonding state from the credits.aleo program.
The cache state will be valid after the caching is successfully finished.
The settlement of the Credits and stCredits of the liquid staking pool must always be based on the **valid** cache state.

```Leo
async transition cache(public start: u32) -> Future;
```

## Admin Transitions

The following transitions can only be called by the admin.
The admin should be an administrator program, which is managed by the Spectre Governance.

#### initialize

Initializes the liquid staking pool.

```Leo
async transition initialize() -> Future;
```

#### register_delegator

Can only be called by the delegator program's `register` transition, to register itself as one delegator for a validator.
The signer must be the admin.
Due to the design of the Aleo native staking mechanism, one delegator can only delegate to one validator at any time. So we need to deploy [multiple delegator programs](delegator.md), each of which is responsible for one validator.

```Leo
async transition register_delegator(public delegator: address, public validator: address) -> Future;
```

#### unregister_delegator

Unregister the delegator and its validator.
The delegator must have neither bonded nor unbonding state.

```Leo
async transition unregister_delegator(public delegator: address) -> Future;
```

#### set_treasury

Set the treasury address of the liquid staking pool.

```Leo
async transition set_treasury(public treasury: address) -> Future;
```

#### set_protocol_fee

Set the protocol fee rate of the liquid staking pool, in percentage (%).

```Leo
async transition set_protocol_fee(public fee: u8) -> Future;
```

#### pause

Pause the liquid staking pool.

```Leo
async transition pause() -> Future;
```

#### unpause

Unpause the liquid staking pool.

```Leo
async transition unpause() -> Future;
```
