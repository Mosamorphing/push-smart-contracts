# PushCoreV2.sol Documentation

- **`pragma solidity ^0.8.0;`**  
  Specifies the Solidity compiler version required to compile this contract. Ensures compatibility and prevents unexpected behavior.

- **`import ...;`**  
  Imports required dependencies and libraries. These may include OpenZeppelin contracts for security and utility functions.

- **`inheritance;`**
- `Initializable`: Allows for the contract to be initialized (a constructor also functions the same way but can't be used for upgradeable contracts)
- `PushCoreStorageV1_5`: Contains storage variables from previous version
- `PausableUpgradeable`: Provides pause/unpause functionality
PushCoreStorageV2: Contains current version storage variables

### Libraries Used
- `SafeMath`: Provides arithmetic operations with safety checks
- `SafeERC20`: Safe wrapper around ERC20 operations

### State Variables
- **`address public owner;`**  
  Declares the owner of the contract. Used for access control and administrative tasks.

### Events
The contract defines multiple events for tracking actions. Some of them include:
- UpdateChannel: Triggered when channel details are updated
- RewardsClaimed: When a user claims rewards
- ChannelVerified: When a channel receives verification
- ChannelVerificationRevoked: When verification is removed,

And many more from lines 50 - 97

### Initialization 
```
function initialize(
    address _pushChannelAdmin,
    address _pushTokenAddress,
    address _wethAddress,
    address _uniswapRouterAddress,
    address _lendingPoolProviderAddress,
    address _daiAddress,
    address _aDaiAddress,
    uint256 _referralCode
) public initializer returns (bool success)
```
Sets up the initial state of the contract including:

- Admin address
- Token addresses
- External contract references
- Fee configurations 
- Base constants

### Access  Control Functions 
```
function onlyPushChannelAdmin() private {
    require(
        msg.sender == pushChannelAdmin,
        "PushCoreV2::onlyPushChannelAdmin: Invalid Caller"
    );
}
```
Internal validation function to restrict function access to the Push channel admin only.

Similar functions exist for:
`onlyGovernance()`
`onlyActivatedChannels(address _channel)`
`onlyChannelOwner(address _channel)`

### Channel Management Functions 
```
function setEpnsCommunicatorAddress(address _commAddress) external
```
This allows an activated channel to add subgraph data, emitting the AddSubGraph event.

```
function setEpnsCommunicatorAddress(address _commAddress) external
```
This sets the address of the EPNS communicator contract. Only callable by the Push channel admin.

```
function updateChannelMeta(
    address _channel,
    bytes calldata _newIdentity,
    uint256 _amount
) external whenNotPaused
```
This updates a channel's metadata:
Updates a channel's metadata:

- Requires caller to be channel owner
- Calculates fee based on update counter
- Transfers PUSH tokens to cover fees
- Updates channel data and emits event

```
function createChannelWithPUSH(
    ChannelType _channelType,
    bytes calldata _identity,
    uint256 _amount,
    uint256 _channelExpiryTime
) external whenNotPaused
```
Creates a new channel:

- Validates deposit amount meets minimum
- Checks channel doesn't already exist
- Validates channel type
- Transfers PUSH tokens
- Calls internal _createChannel function


```
function _createChannel(
    address _channel,
    ChannelType _channelType,
    uint256 _amountDeposited,
    uint256 _channelExpiryTime
) private
```

Internal function to handle private channel creation:

- Calculates fee distribution
- Sets channel state to active (1)
- Records contribution and weight
- Sets channel type and timestamps
- Handles time-bound channel expiry
- Subscribes channel to relevant channels

```
function destroyTimeBoundChannel(address _channelAddress)
    external
    whenNotPaused
```

Destroys a time-bound channel after expiry:

- Validates channel is time-bound type
- Checks caller is owner or admin (with timelock)
- Calculates refundable amount
- Updates pool funds
- Handles unsubscriptions
- Deletes channel data

```
function createChannelSettings(
    uint256 _notifOptions,
    string calldata _notifSettings,
    string calldata _notifDescription,
    uint256 _amountDeposited
) external
```

Sets notification options for a channel:

- Caller must be an activated channel
- Validates sufficient deposit
- Stores notification settings
- Updates protocol fees
- Emits settings event

```
function deactivateChannel() external whenNotPaused
```
Allows channel owner to deactivate their channel:

- Changes channel state to deactivated (2)
- Calculates refundable amount (keeping minimum contribution)
- Updates pool funds and weights
- Transfers refundable amount to caller
- Emits deactivation event

```
function reactivateChannel(uint256 _amount) external whenNotPaused
```
Allows reactivation of a deactivated channel:

- Validates sufficient deposit
- Verifies channel is in deactivated state
- Transfers PUSH tokens
- Distributes between pool funds and fees
- Updates channel state to active (1)

```
function blockChannel(address _channelAddress) external whenNotPaused
``` 
Allows admin to block a channel:

- Only callable by pushChannelAdmin
- Changes channel state to blocked (3)
- Moves funds from pool to protocol fees
- Adjusts channel weight
- Decrements channel count
- Emits blocking event

### Channel Verification

```
function getChannelVerfication(address _channel)
    public
    view
    returns (uint8 verificationStatus)
```
Returns channel verification status:

- 0: Not verified
- 1: Primary verification (by admin)
- 2: Secondary verification (by verified channel)

```function verifyChannel(address _channel) public
```
Verifies a channel:

- Verifier must be verified themselves
- Target must be unverified or verifier is admin
- Updates verification status

```function unverifyChannel(address _channel) public
```
Removes verification from a channel:

- Only callable by the verifier or admin
- Resets verification status
- Emits revocation event

### Staking and Rewards

```
function initializeStake() external
``` 
Initializes the staking system:

- Can only be called once
- Sets genesis epoch block
- Seeds pool with 1 PUSH token

```
function stake(uint256 _amount) external
``` 
Allows users to stake PUSH tokens:

- Calls internal _stake function
- Emits staking event

```
function _stake(address _staker, uint256 _amount) private
```
Internal staking implementation:

- Calculates current epoch and weights
- Transfers tokens to contract
- Updates user and total staked amounts
- Adjusts user and total weights

```
function unstake() external
```
Allows unstaking of tokens:

- Requires minimum staking period
- Harvests all pending rewards
- Transfers staked amount back to user
- Adjusts user and total weights
- Resets user stake data


```
function harvestAll() public
``` 
Harvests all pending rewards:

- Calculates current epoch
- Calls internal harvest function
- Transfers rewards to caller

```
function harvestPaginated(uint256 _tillEpoch) external
``` 
Allows paginated reward harvesting:
- Limits harvesting to specified epoch
- Calls internal harvest function
- Transfers rewards to caller

```
function harvest(address _user, uint256 _tillEpoch)
    internal
    returns (uint256 rewards)
```

Internal reward harvesting logic:
- Resets token holder weight
- Adjusts user and total stakes
- Calculates epochs for harvesting
- Validates epoch boundaries
- Sums rewards across epochs
- Updates user claimed rewards
- Updates last claimed block
- Emits harvesting event

```
function _adjustUserAndTotalStake(address _user, uint256 _userWeight)
    internal
```
Adjusts user and total staked weights:

- Calculates current epoch
- Sets up epoch rewards and weights
- Handles first-time stakers
- Handles repeat stakers in same epoch
- Handles repeat stakers in different epochs
- Updates last staked block if needed

```
function _setupEpochsRewardAndWeights(
    uint256 _userWeight,
    uint256 _currentEpoch
) private
```
Sets up rewards and weights for epochs:

- Initializes epoch-based rewards
- Distributes available rewards
- Updates epoch tracking variables
- Initializes or updates total weights for epochs


### Incentivized Chat Functions
```
function handleChatRequestData(
    address requestSender,
    address requestReceiver,
    uint256 amount
) external
```
Handles incentivized chat request data:

- Validates caller is communicator contract
- Calculates fee distribution
- Updates user funds and protocol fees
- Emits request event

```
function claimChatIncentives(uint256 _amount) external
```
Allows users to claim chat incentives:

- Validates user has sufficient funds
- Updates user balance
- Transfers tokens to user
- Emits claim event