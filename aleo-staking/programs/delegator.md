- [Source code](https://github.com/spectrehq/spectre/tree/main/packages/leo/spectre_v1/delegator)
- Deployed program (coming soon)

Delegator is a program that help users to delegate their Credits tokens to validators on behalf of the liquid staking pool.

Since the Aleo native staking mechanism only allows one delegator to delegate to one validator at any time, we cannot let the liquid staking pool program (stCredits) delegate to multiple validators at the same time. Therefore, we need several separate programs to handle the one-to-one delegation process. We can deploy the delegator program multiple copies with different program names (actually add different suffixes). And each delegator copy should `register` itself and its validator to the liquid staking pool program. Once registered, the delegator can `bond` (delegate) Credits to its validator. It is important to note as well that the `unbond` will be done by the liquid staking pool program, not the delegator program. This is because the liquid staking program is the one that holds the Credits tokens on behalf of the users, and all the delegator programs always use the liquid staking program address as the withdrawal address (see struct `Withdraw` in credits.aleo).

## Operator Transitions

The following transitions can only be called by the operator.
The operator account should be controlled by the operator daemon service, which is maintained by the Spectre team and is managed by the admin (or the Spectre Governance).

#### bond

Bond (delegate) the amount of Credits to the validator.
It will call the `bond` transition in the liquid staking program, and then call the `bond_public` transition in the credits.aleo program.

```Leo
async transition bond(public validator: address, public amount: u64) -> Future;
```

## Admin Transitions

The following transitions can only be called by the admin.
The admin should be an administrator program, which is managed by the Spectre Governance.

#### register

Register the delegator program and its validator to the liquid staking program.
It will call the `register_delegator` transition in the liquid staking program.

```Leo
async transition register(public validator: address) -> Future;
```
