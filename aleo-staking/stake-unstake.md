## Stake

Staking is the key element of the Aleo network. Validators stake Credits, validate transactions and participate in consensus to add new blocks, in order to keep the Aleo network operational. In exchange for this service validators earn fees and rewards.

AleoStaking helps users gain access to becoming a staking provider without the overhead of running validators. We achieve this by allowing users to deposit Credits, these deposits are then distributed to validators who run Aleo nodes on behalf of AleoStaking. As AleoStaking runs validators rewards will accrue to stakers.

As a liquid staking provider AleoStaking extends this one step further by returning users with a stCredits token which represents their stake and is usable similarly to stCredits throughout ZeFi.

AleoStaking's stCredits token is an index token, this means that it represents the users ownership of all the underlying Credits that the AleoStaking protocol owns. As the protocol earns fees and rewards, the stCredits owned by a user will reflect ownership of more and more Credits.

## Unstake

stCredits can be redeemed for Credits at any time. The redemption process is called unstaking. When a user unstakes their stCredits, the protocol will burn the stCredits and return the user the underlying Credits.

#### Delayed Unstaking

Like Aleo's native staking, liquid staking with AleoStaking requires a delay before the staked Credits can be withdrawn. This delay is called the unbonding period. The unbonding period is set to 360 blocks. This means that after a user initiates an unstake request, they will have to wait for **around** 360 blocks before they can withdraw their staked Credits.

#### Instant Unstaking

AleoStaking also offers an instant unstaking feature. This feature allows users to unstake their staked Credits instantly. However, this feature comes with a fee. The fee is determined by the governance of the protocol.
