# PushCore V2 Overview

PushCore V2 is a core component of the Push (formerly EPNS - Ethereum Push Notification Service) protocol. This contract handles: 

- Channel Management (Creation, Deactivation, Verification, Blocking)
- Staking & Rewards (Users can stake PUSH tokens to earn rewards)
- Governance & Admin Controls (Fee adjustments, contract pausing)
- Incentivized Chat Requests (Handling fees for chat requests)

## Key features and functions

### A. Channel Management

1. Channel Creation:
- Users can create channels by depositing PUSH tokens.
- Different channel types:
    - `InterestBearingOpen`
    - `InterestBearingMutual`
    - `TimeBound` (expires after a set time)
    - `TokenGaited`
- Fees:
    - Minimum deposit: `50 PUSH` (configurable via governance
    - Fees are split between `CHANNEL_POOL_FUNDS` and `PROTOCOL_POOL_FEES`.

2. Channel Updates & Meta Changes:
- Channel owners can update metadata (name, logo, description).
- Each update requires an increasing fee (`50 PUSH * updateCount`).

3. Channel Deactivation & Reactivation:
- Channels can be deactivated, refunding most of the staked tokens (minus `MIN_POOL_CONTRIBUTION`).
- Reactivation requires a new deposit (`≥50 PUSH`).

4. Time-Bound Channel Destruction
- Time-bound channels can be destroyed after expiry.
- If destroyed by the owner, they get a refund.
- If destroyed by admin (after 14 days of expiry), funds go to `PROTOCOL_POOL_FEES`.

5. Channel Verification
- Primary Verification: Done by `pushChannelAdmin`.
- Secondary Verification: Verified channels can verify others.
- Verification can be revoked by the verifier or admin.


### B. Staking & Rewards

1. Staking Mechanism:
- Users stake PUSH tokens to earn rewards.
- Rewards are distributed per epoch (time-based intervals).
- Staked tokens contribute to user weight, which determines rewards.

2. Reward Calculation:
- Rewards = `(userStakedWeight * epochRewards) / totalStakedWeight`.
- Users can harvest rewards manually or via paginated claims.

3. Governance Staking:
- The DAO (governance) can also stake and claim rewards.


### C. Governance & Admin Controls

1. Fee Adjustments:
- `pushChannelAdmin` can modify:
    - `FEE_AMOUNT` (default: `10 PUSH`)
    - `MIN_POOL_CONTRIBUTION` (default: `50 PUSH`)
    - `ADD_CHANNEL_MIN_FEES` (minimum channel creation fee)

2. Contract Pausing:
- The contract can be paused/unpaused by governance.

3. Admin Transfer:
- `pushChannelAdmin` can transfer control to a new address.


### D. Incentivized Chat Requests

- Users can send incentivized chat requests to others. 
- A fee (10 PUSH) is charged, split between:
    - Receiver (Celeb user, e.g., influencers)
    - Protocol Pool Fees
- Celeb users can claim their accumulated PUSH tokens.


## Security & Access Control

- Role-Based Permissions:
    - `onlyPushChannelAdmin()` → For critical admin functions
    - `onlyGovernance()` → For fee adjustments and pausing.
    - `onlyChannelOwner()` → For channel-specific updates.
- Pausable → Emergency stop mechanism.
- Input Validation → Checks for valid amounts, channel states, and expiry times.


## Events

Key events emitted for tracking:
- AddChannel → New channel created.
- UpdateChannel → Channel metadata updated.
- RewardsClaimed → User harvested rewards.
- ChannelVerified → Channel verification status changed.
- IncentivizeChatReqReceived → Chat request activated.