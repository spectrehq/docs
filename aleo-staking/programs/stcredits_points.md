- [Source code](https://github.com/spectrehq/spectre/tree/main/packages/leo/spectre_v1/stcredits_points)
- Deployed program (coming soon)

AleoStaking incentivizes users to lock their stCredits into the AleoStaking Points program by rewarding them with [Points](../../spectre/points.md). These points are earned based on the amount of stCredits locked and the duration of the lock-up period. The longer the lock-up period and the larger the amount of stCredits locked, the higher the points accumulates.

It's worth noting that the AleoStaking Points is one implementation of the [Spectre Points](../../spectre/points.md) in the AleoStaking product. That is to say, the portion of the points that users earn from AleoStaking is a part of the total points of Spectre.

## Mappings, Structs and Records

All the mappings can be read publicly, either by the user or by the programs.

#### total_supply

Returns the total supply of AleoStaking Points.
The only valid key is `0u8`.

```Leo
mapping total_supply: u8 => u128;
```

#### account

Returns the amount of AleoStaking Points that the account owns.

```Leo
mapping account: address => u128;
```

#### states

Returns the state of the user.
The state includes the total locked stCredits, the block height at which the last settlement was made, the inviter, and the inviter of the inviter.

```Leo
struct UserState {
    stcredits: u64, // the total locked stcredits
    height: u32, // the block height at which the last settlement was made
    inviter: address,
    inviter_of_inviter: address,
}

mapping states: address => UserState;
```

#### inviters

Return the user address by the invite code.

```Leo
mapping inviters: u32 => address;
```

#### invite_codes

Return the invite code by the user address.

```Leo
mapping invite_codes: address => u32;
```

#### invite_code_counter

Returns the invite code incrementer which is used to generate the invite code.

```Leo
mapping invite_code_counter: u8 => u32;
```

#### paused

Whether the program is paused or not.

```Leo
mapping paused: u8 => bool;
```

## Transitions

#### lock

Lock user's stCredits into the program to get points, with an optional invite code.
For a new user, the `invite_code` points to the user's inviter.
And if `invite_code` is empty (zero), the caller's inviter and the inviter of inviter will be both none.
If the user is not new, `invite_code` must be empty (zero).

```Leo
async transition lock(public stcredits: u64, public invite_code: u32) -> Future;
```

#### unlock

Unlock user's stCredits and settle points.

```Leo
async transition unlock(public stcredits: u64) -> Future;
```

#### settle

Settle points for the user at current block height.

```Leo
async transition settle() -> Future;
```

## Admin Transitions

The following transitions can only be called by the admin.
The admin should be an administrator program, which is managed by the Spectre Governance.

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
