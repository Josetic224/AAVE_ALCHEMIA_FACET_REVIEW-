| Client         | Gotchiverse Realm Diamond               |
| :------------- | :---------------------------------------------- |
| Title          | Smart Contract Audit Report                     |
| Target         | AlchemicaFacet.sol                            |
| Version        | 1.0.0                                           |
| Author         | [Josetic224](https://github.com/Josetic224) |
| Classification | Public                                          |
| Date Created   | April 7, 2025                              |

## Table of contents

- <a href="#intro"> 1. INTRODUCTION</a>

  - <a href="#Disclaim"> 1.1 Disclaimer</a>
  - <a href="#About"> 1.2 About Me </a>
  - <a href="#Skills"> 1.3 Skills</a>
  - <a href="#links"> 1.4 Link</a>
  - <a href="#scope"> 1.7 Scope</a>
  - <a href="#roles"> 1.8 Roles</a>
  - <a href="#overview"> 1.9 System Overview</a>

- <a href="#review"> 2.0 CONTRACT REVIEW</a>

- <a href="#findings"> 3.0 FINDINGS</a>

  - <a href="#Qanalysis"> 3.1 Qualitative Analysis</a>
  - <a href="#summary"> 3.2 Summarys</a>
  - <a href="#recom"> 3.2 Recommendations</a>

- <a href="#conclusion"> 4.0 CONCLUSION</a>

<h2 id="intro">1.0 INTRODUCTION </h2>

### <h3 id="Disclaim">1.1 Disclaimer <h3>

This document is a technical review I conducted as a smart contract developer, focusing on the AlchemicaFacet of the Aavegotchi Realm Diamond. It's not a formal audit, but rather an independent walkthrough meant to highlight how the contract works, potential areas for improvement, and anything I found interesting or worth noting.

The review is based solely on the source code that was available at the time. It doesn’t include formal security guarantees, full test coverage, or exhaustive verification. My goal here is to provide helpful insights and contribute to understanding the contract — not to certify it as secure or production-ready.

As always, anyone building on or interacting with this contract should do their own due diligence and testing.

### <h3 id="About">1.2 🚀 About Me <h3>

I am Josetic, a smart contract developer still figuring things out one repo at a time. I'm currently interning at Web3bridge, where I’m learning by building, breaking stuff, and writing better solidity code.
This review is part of my learning process — just me trying to understand how things work in the wild and getting more comfortable reading real-world contracts.

Currently interested in architecture, security and scalablity of blockchain protocols.

### <h3 id="Skills">1.3 🛠 Skills <h3>

Solidity, Diamond Standard, Foundry, Hardhat, Wagmi, Ethers.js, React, Next js, Express, Bash, Typescript.

### <h3 id="links">1.4 🔗 Links <h3>

[![portfolio](https://img.shields.io/badge/my_portfolio-000?style=for-the-badge&logo=ko-fi&logoColor=white)](https://github.com/Josetic224)
[![X](https://img.shields.io/badge/X-000000?style=for-the-badge&logo=x&logoColor=white)](https://x.com/yoseph_Eth)

### <h3 id="Cpg">1.5 Aavegotchi's AlchemicaFacet<h3>

Explain context

### <h3 id="Gbd">1.6 AlchemicaFacet<h3>

The AlchemicaFacet is a crucial component of the Aavegotchi Realm Diamond, designed to manage and interact with the Alchemica tokens (FUD, FOMO, ALPHA, KEK) that fuel various aspects of the Gotchiverse. These tokens serve as in-game resources that are key to upgrading land, building structures, and enhancing gameplay mechanics within the ecosystem.

This facet is responsible for the minting, distribution, and burning of Alchemica tokens, enabling players to collect and use them in the game. Through different in-game interactions, players can channel, harvest, and spend Alchemica to improve their land, increase their yields, and unlock new capabilities.

The AlchemicaFacet plays an integral role in the Gotchiverse economy, ensuring the secure, transparent, and fair handling of these resources while driving player engagement and in-game progression.

### <h3 id="scope">1.7 Scope <h3>

_(**Table: 1.7**: AlchemicaFacet Audit Scope)_
| Files in scope | SLOC |
| :-------- | :------- |
| Contracts: 1 | |
| `AlchemicaFacet.sol` | `427` |
| | |
| Imports: 8 | |
| `AppStorage.sol` | `161` |
| `RealmFacet.sol` | `353` |
| `LibRealm.sol` | `260` |
| `LibMeta.sol` | `32` |
| `VRFCoordinatorV2Interface.sol` | `X` |
| `LibAlchemica.sol` | `351` |
| `LibSignature.sol` | `34` |

### <h3 id="overview"> 1.9 System Overview <h3>

On a high-level overview of the AlchemicaFacet Contract, it performs these 5 things

- #### Parcel Interaction and Surveying
The contract enables Players to manage their parcels by surveying them to track the available Alchemica. Surveys allow Players to monitor resource accumulation, ensuring they can efficiently plan harvests and maximize the value of their parcels.

- #### Alchemica Harvesting and Claiming
It provides the ability for Players to harvest and claim Alchemica from their parcels. This feature supports both individual and batch claims, allowing users to gather their resources in a way that suits their strategy and needs.

- #### Channeling Resources to Aavegotchis
A key feature of the contract is its ability to channel harvested Alchemica(tokens) to Aavegotchis. This allows users to boost their Aavegotchis' stats or abilities, adding a layer of strategic depth to the use of Alchemica.

- #### Resource Distribution and Transfer
The contract facilitates the distribution of Alchemica to multiple addresses, making it easier for users to manage their rewards, transfer resources to others, or handle bulk transactions.

- #### Admin Controls for System Management
The contract includes administrative functions that allow for the management of system parameters such as claim limits and transfer restrictions. These controls ensure the system remains balanced, secure, and adaptable to changes.


##  <h2 id="review"> 2.0 CONTRACT REVIEW <h2>

The contract inherits from Modifiers Contract located in **AppStorage.sol**

`Modifiers` is  is a base contract that implements various access control and state validation modifiers used throughout the codebase. In the context of **AlchemicaFacet.sol**, these modifiers serve critical security and functionality roles.

**AlchemicaFacet.sol** inherits from Modifiers to use these access control and state validation mechanisms

```bash
  contract AlchemicaFacet is Modifiers
```

_Since this is contract is part of a Diamond, all state variables are stored in the shared AppStorage struct defined in **AppStorage.sol**._

#### The AlchemicaFacet relies on these AppStorage variables



`alchemicaAddresses;`   The contract uses these addresses to interact with different Alchemica tokens (FUD, FOMO, ALPHA, KEK). For example, interacting with the FUD token

```bash
  address[4] alchemicaAddresses;
```

`totalAlchemicas` This variable has a 2D array where each row corresponds to a specific Alchemica type (FUD, FOMO, ALPHA, KEK, etc.) and each column corresponds to a specific measurement (perhaps total staked, remaining, etc.).

```bash
  uint256[4][5] totalAlchemicas;
```

`boostMultipliers` This array holds multipliers that affect the amount of Alchemica harvested or rewarded based on certain boosts (e.g., specific boosts for different types of Alchemica).

```bash
uint256[4] boostMultipliers
```

`greatPortalCapacity`  This array tracks the capacity of the "Great Portal" for each type of Alchemica, likely to restrict how much can be transferred or harvested in specific game events.
This would be used in **AlchemicaFacet.sol**  to ensure players cannot exceed the portal's capacity when transferring or harvesting Alchemica. 

` vrfCoordinator` The address of the Chainlink VRF (Verifiable Random Function) Coordinator, which is used to generate secure random numbers for various game mechanics.
The **AlchemicaFacet.sol** contract calls this address to request randomness for game events like surveys or determining randomness in the Alchemica harvesting process.

```bash
address vrfCoordinator
```

`linkAddress` The address of the LINK token, used for paying fees to Chainlink's VRF service.
This is used in **AlchemicaFacet.sol** to send LINK tokens to pay for VRF requests.

```bash
address linkAddress
```

` requestConfig` This struct holds configuration details for VRF requests, such as the gas limit and other parameters for requesting randomness.
In **AlchemicaFacet.sol**, The requestConfig is used to set up and manage how Chainlink VRF randomness is requested and configured.

```bash
RequestConfig requestConfig
```
`vrfRequestIdToTokenId` This mapping associates VRF request IDs with realm or token IDs. This is useful for tracking the request and linking it to a specific parcel or realm.
When a randomness request is made, the contract(**AlchemicaFacet.sol**) can track which token or parcel the randomness is related to by storing the mapping.

```bash
mapping(uint256 => uint256) vrfRequestIdToTokenId;
```
`vrfRequestIdToSurveyingRound` This mapping links VRF request IDs to the surveying round for which the randomness is needed.  In **AlchemicaFacet.sol** This is useful for tracking which survey round a randomness request pertains to, ensuring the right random value is used for the correct round.

```bash
mapping(uint256 => uint256) vrfRequestIdToSurveyingRound
```

`backendPubKey`  The public key for the backend, likely used for verifying off-chain signatures or interactions. This key could be used for ensuring that off-chain data or transactions are signed by the backend and verified on-chain.

```bash
bytes backendPubKey
```


`gameManager` The address of the game manager or controller, who is likely responsible for overseeing and controlling important aspects of the game, such as access control. In **AlchemicaFacet.sol** The contract will check if the sender is the game manager for certain privileged actions.

```bash
address gameManager
```


`installationsDiamond` The address of the Installations Diamond, likely used for managing game installations or upgrades.
In **AlchemicaFacet.sol**, this address is used to call into the Installations Diamond for managing installations and updates in the game.

```bash
address installationsDiamond
```

`gltrAddress` The address of the Installations Diamond, mostly used for managing game installations or upgrades.
In **AlchemicaFacet.sol**, this address is used to call into the Installations Diamond for managing installations and updates in the game.

```bash
address gltrAddress;
```

`tileDiamond` The address of the Tile Diamond, which likely manages in-game tile-related functionalities (e.g., managing land or tiles for players).
In **AlchemicaFacet.sol** This address is used to interact with tile-related features, such as purchasing or upgrading tiles

```bash
address tileDiamond;
```

`aavegotchiDiamond` The address of the Aavegotchi Diamond, which likely manages interactions with the Aavegotchi protocol (e.g., summoning Aavegotchis or performing actions related to them).
In **AlchemicaFacet.sol** This address is used to interface with the Aavegotchi Diamond for game mechanics like summoning or upgrading Aavegotchis.

```bash
address aavegotchiDiamond;
```

`lastClaimedAlchemica`  This mapping tracks the last time an Alchemica claim was made for a specific parcel. It is used to ensure that players can only claim rewards after a certain time.
In **AlchemicaFacet.sol** This mapping is checked when determining if a claim is allowed, and it is updated after a successful claim.

```bash
mapping(uint256 => uint256) lastClaimedAlchemica;
```

`surveying` This boolean indicates whether a parcel is currently being surveyed for Alchemica gathering.
In **AlchemicaFacet.sol** This is checked to see if the parcel is in a valid surveying state before allowing certain actions to proceed.
```bash
bool surveying;
```

`currentRound`  This variable tracks the current round for the surveying and Alchemica collection process.
In **AlchemicaFacet.sol** This is used to manage the progression of rounds for harvesting Alchemica, allowing the game to handle multiple rounds of collection.

```bash
uint256 currentRound;
```

`altarId`  This variable stores the ID of the altar associated with a particular realm or parcel.
In **AlchemicaFacet.sol** This ID is used to access or interact with the altar and its associated features.

```bash
uint256 altarId;
```
`alchemicaRemaining`  This array tracks the remaining amounts of Alchemica for a specific parcel.
In **AlchemicaFacet.sol** This is used to determine how much Alchemica can still be harvested or claimed for a given parcel.
```bash
uint256[4] alchemicaRemaining;
```
`roundBaseAlchemica`  This mapping stores base Alchemica amounts for each round.
In **AlchemicaFacet.sol** It is used for calculations when determining how much Alchemica should be awarded or transferred for a specific round.
```bash
mapping(uint256 => uint256[]) roundBaseAlchemica;
```
`roundAlchemica`  This mapping tracks Alchemica values collected during specific rounds.
In **AlchemicaFacet.sol** This is used when collecting or transferring Alchemica during a particular round.
```bash
mapping(uint256 => uint256[]) roundAlchemica;
```

`alchemicaHarvestRate`   This array holds the rate at which Alchemica is harvested for each type (FUD, FOMO, ALPHA, KEK).
In **AlchemicaFacet.sol** It is used to adjust the amount of Alchemica collected or rewarded per round or survey.
```bash
uint256[4] alchemicaHarvestRate;
```

The following functions are part of **Aavegotchi's AlchemicaFacet contract** and are all carefully and thoroghly intertwined for optimum performance.

The flow:

- Events
- Getter
- Internal/Private
- External

### 2.1 Events
`event StartSurveying(uint256 _realmId, uint256 _round);`

  **Purpose:** Signals the beginning of surveying a parcel.

  **Use Cases:**
  Frontend can listen for this to trigger animations or enable interactions.
  Subgraphs can use this to index surveying rounds.

 `event ChannelAlchemica(uint256 indexed _realmId, uint256 indexed _gotchiId, uint256[4] _alchemica, uint256 _spilloverRate, uint256 _spilloverRadius);`

  **Purpose:**  Emits when a Gotchi channels Alchemica from a realm.

  **Details:**

  - alchemica: array of amounts for 4 types (FUD, FOMO, ALPHA, KEK).

- spilloverRate and _spilloverRadius: used for in-game logic like where spillover lands.
  
 **Strength:**
 - Already indexed by both _realmId and _gotchiId which is great for filtering.
  
`event ExitAlchemica(uint256 indexed _gotchiId, uint256[] _alchemica);`
**Purpose:** Indicates when a Gotchi exits a realm or session, releasing or transferring Alchemica.

 **Usefulness**

  - It lets the frontend or subgraph update the Gotchi's state.

- `_alchemica` isn't fixed size, which makes it flexible for future changes but less efficient to decode.

`event SurveyingRoundProgressed(uint256 indexed _newRound);`
**Purpose:** Indicates when a Gotchi exits a realm or session, releasing or transferring Alchemica.

 **Usefulness**

  - Tied to game-wide pacing or epoch-like mechanics.

- Helps UIs and subgraphs shift their state and calculations.

`event TransferTokensToGotchi(address indexed _sender, uint256 indexed _gotchiId, address _tokenAddresses, uint256 _amount);;`

**Purpose:** Indicates that a wallet sent tokens to a Gotchi (in-Gotchi wallet).

 **Fields:**

  - _sender: who sent the token.

  - _tokenAddresses: which token was sent (ERC20).

  - _amount: how much was sent.

  - Helps UIs and subgraphs shift their state and calculations.

**Use Cases:**

- UI might update a Gotchi's in-game balance.

- On-chain actions like funding spells, boosts, etc.
  
### Getter Functions
 
#### 2.2 isSurveying()

The `isSurveying` function checks whether a specific parcel (identified by `_realmId`) is currently in the surveying state.

Breakdown:
`s.parcels[_realmId]`: Accesses the parcel data for the given `_realmId` from the storage (`s`) This is coming from AppStorage.sol.

`.surveying`: Retrieves the `surveying` boolean flag from the parcel's data.

Returns: `true` if the parcel is currently surveying; `false` otherwise.

```bash
     function isSurveying(uint256 _realmId) external view returns (bool) {
    return s.parcels[_realmId].surveying;
  }
```

#### 2.3 startSurveying()
The `startSurveying` function allows the owner of a specific parcel to begin the surveying process for that parcel, with several checks to ensure the operation can proceed.

**Breakdown**

**Access Control** (`onlyParcelOwner(_realmId)`): Ensures that only the `parcel owner` can start surveying.

**gameActive Modifier:** Ensures that the `game` is active before proceeding.

**Round Validation:** The first `require` statement checks that the `currentRound` of the parcel is less than or equal to the `surveyingRound`. This ensures that the surveying can only start once the round has been released.

**Altar Requirement:** The second `require` ensures that the parcel has an `altarId` (indicating that an altar is equipped) before surveying can begin.

**Surveying Check:** The third `require` ensures that the parcel is not already being surveyed.

**Set Surveying State:** Sets `surveying` = true for the parcel, indicating that it is now in the surveying state.

**Random Number Drawing:** The function calls `drawRandomNumbers` to generate `random numbers `associated with the survey for the `current round`.

**Emit Event:** The function emits a `StartSurveying` event to signal that the surveying process has started for the parcel.

```bash
  function startSurveying(uint256 _realmId) external onlyParcelOwner(_realmId) gameActive {
    //current round and surveying round both begin at 0.
    //after calling VRF, currentRound increases
    require(s.parcels[_realmId].currentRound <= s.surveyingRound, "AlchemicaFacet: Round not released");
    require(s.parcels[_realmId].altarId > 0, "AlchemicaFacet: Must equip Altar");
    require(!s.parcels[_realmId].surveying, "AlchemicaFacet: Parcel already surveying");
    s.parcels[_realmId].surveying = true;
    // do we need to cancel the listing?
    drawRandomNumbers(_realmId, s.parcels[_realmId].currentRound);

    emit StartSurveying(_realmId, s.parcels[_realmId].currentRound);
  }
```

#### 2.4 drawRandomNumbers()
This function requests random numbers from a VRF (Verifiable Random Function) service for a specified parcel and surveying round.

**Breakdown**

**Request Random Numbers:** The function calls _**VRFCoordinatorV2Interface(s.vrfCoordinator).requestRandomWords()**_ to request random numbers. This requires the following parameters:

- **keyHash:** A unique identifier for the VRF service.

- **subId**: The subscription ID for VRF service.

- **requestConfirmations:** The number of confirmations required before the request is considered valid.

- **callbackGasLimit:** The gas limit for the callback function.

- **numWords:** The number of random numbers to be returned.

**Mapping VRF Request:**

- **s.vrfRequestIdToTokenId**: Maps the `requestId` to the _realmId, tracking which parcel the random numbers are being requested for.

- **s.vrfRequestIdToSurveyingRound:** Maps the `requestId` to the `_surveyingRound`, linking the random numbers request to the specific surveying round.
  
```bash
  function drawRandomNumbers(uint256 _realmId, uint256 _surveyingRound) internal {
    // Will revert if subscription is not set and funded.
    uint256 requestId = VRFCoordinatorV2Interface(s.vrfCoordinator).requestRandomWords(
      s.requestConfig.keyHash,
      s.requestConfig.subId,
      s.requestConfig.requestConfirmations,
      s.requestConfig.callbackGasLimit,
      s.requestConfig.numWords
    );
    s.vrfRequestIdToTokenId[requestId] = _realmId;
    s.vrfRequestIdToSurveyingRound[requestId] = _surveyingRound;
  }
```

#### 2.6 getAlchemicaAddresses()
This function returns the array of alchemica token addresses.

It returns `s.alchemicaAddresses`, an array containing the addresses of the alchemica tokens.

```bash
 function getAlchemicaAddresses() external view returns (address[4] memory) {
    return s.alchemicaAddresses;
  }
```

#### 2.7 getTotalAlchemicas()
This function returns a two-dimensional array that provides details about the total alchemica present in the system.
It returns `s.totalAlchemicas`, a 2D array where each entry corresponds to an alchemica value.

```bash
   function getTotalAlchemicas() external view returns (uint256[4][5] memory) {
    return s.totalAlchemicas;
  }
```

#### 2.8 getRealmAlchemica()
This function returns an array with details about the remaining alchemica in a specific parcel.
The function takes `_realmId` as a parameter to query details for the specific parcel.
It returns `s.parcels[_realmId].alchemicaRemaining`, an array containing the remaining alchemica for the parcel.


```bash
   function getRealmAlchemica(uint256 _realmId) external view returns (uint256[4] memory) {
    return s.parcels[_realmId].alchemicaRemaining;
  }
```
#### 2.9 getParcelCurrentRound()
This function is a **view** function that allows users to query the current round number associated with a particular parcel. The ``currentRound`` is a value stored in the contract's state for the parcel identified by ``_realmId``. It returns this value, giving information about the current round of alchemica surveying for that specific parcel.

```bash
/// @notice Query details about the remaining alchemica in a parcel
/// @param _realmId The identifier of the parcel to query
/// @return output_ An array containing details about each remaining alchemica in the parcel
function getParcelCurrentRound(uint256 _realmId) external view returns (uint256) {
    return s.parcels[_realmId].currentRound;
}
```

#### 2.10 progressSurveyingRound()
This function allows the owner (probably the contract's admin or an account with special privileges) to progress to the next surveying round by incrementing the `surveyingRound` counter. When the round is incremented, it triggers the `SurveyingRoundProgressed` event to notify the system that the surveying round has progressed.

```bash
/// @notice Allow the diamond owner to increment the surveying round
function progressSurveyingRound() external onlyOwner {
    s.surveyingRound++;
    emit SurveyingRoundProgressed(s.surveyingRound);
}
```

#### 2.11 getRoundAlchemica()

This view function allows users to query the **alchemica** gathered during a specific surveying round for a given parcel (`_realmId`). The `_roundId` refers to a specific round, and the function returns an array representing the quantities of alchemica gathered during that round.

``` bash
/// @notice Query details about all alchemica gathered in a surveying round in a parcel
/// @param _realmId Identifier of the parcel to query
/// @param _roundId Identifier of the surveying round to query
/// @return output_ An array representing the numbers of alchemica gathered in a round
function getRoundAlchemica(uint256 _realmId, uint256 _roundId) external view returns (uint256[] memory) {
    return s.parcels[_realmId].roundAlchemica[_roundId];
}
```

#### 2.12 getRoundBaseAlchemica()

Similar to the previous function, this one queries the base alchemica (perhaps a specific type of alchemica) gathered during a specific surveying round (`_roundId`) for a particular parcel (`_realmId`). It returns an array of base alchemica gathered.

```bash
/// @notice Query details about the base alchemica gathered in a surveying round in a parcel
/// @param _realmId Identifier of the parcel to query
/// @param _roundId Identifier of the surveying round to query
/// @return output_ An array representing the numbers of base alchemica gathered in a round
function getRoundBaseAlchemica(uint256 _realmId, uint256 _roundId) external view returns (uint256[] memory) {
    return s.parcels[_realmId].roundBaseAlchemica[_roundId];
}
```

#### 2.13 setVars()

 This is an owner-only function that allows the owner to set a variety of important state variables for the contract. These variables include data related to **alchemica**, **boost multipliers**, **portal capacity**, the **addresses** for various contracts (e.g., _installations_, _Chainlink VRF_, _Aavegotchi_, etc.), and backend-related information. These parameters allow for the customization and configuration of the system.

 ```bash
function setVars(
    uint256[4][5] calldata _alchemicas,
    uint256[4] calldata _boostMultipliers,
    uint256[4] calldata _greatPortalCapacity,
    address _installationsDiamond,
    address _vrfCoordinator,
    address _linkAddress,
    address[4] calldata _alchemicaAddresses,
    address _gltrAddress,
    bytes memory _backendPubKey,
    address _gameManager,
    address _tileDiamond,
    address _aavegotchiDiamond
) external onlyOwner {
    for (uint256 i; i < _alchemicas.length; i++) {
        for (uint256 j; j < _alchemicas[i].length; j++) {
            s.totalAlchemicas[i][j] = _alchemicas[i][j];
        }
    }
    s.boostMultipliers = _boostMultipliers;
    s.greatPortalCapacity = _greatPortalCapacity;
    s.installationsDiamond = _installationsDiamond;
    s.vrfCoordinator = _vrfCoordinator;
    s.linkAddress = _linkAddress;
    s.alchemicaAddresses = _alchemicaAddresses;
    s.backendPubKey = _backendPubKey;
    s.gameManager = _gameManager;
    s.gltrAddress = _gltrAddress;
    s.tileDiamond = _tileDiamond;
    s.aavegotchiDiamond = _aavegotchiDiamond;
}
 ```

 #### 2.14 setTotalAlchemicas()
 This function is only accessible by the owner and allows the owner to set the total amount of alchemicas across different categories (4 types of alchemicas, 5 categories in total) in the contract's state.

The `totalAlchemicas` array is updated with the values passed in the `_totalAlchemicas` parameter (which is a 2D array of size 4x5).

The function loops over each element and updates the state variable `s.totalAlchemicas`.

```bash
function setTotalAlchemicas(uint256[4][5] calldata _totalAlchemicas) external onlyOwner {
    for (uint256 i; i < _totalAlchemicas.length; i++) {
      for (uint256 j; j < _totalAlchemicas[i].length; j++) {
        s.totalAlchemicas[i][j] = _totalAlchemicas[i][j];
      }
    }
}
```

#### 2.15 getAvailableAlchemica()
This view function returns the available alchemica for a given parcel (`_realmId`). It uses a helper library `LibAlchemica` to query the available alchemica for the parcel.
The function loops through the 4 different types of alchemica and calls the library function `getAvailableAlchemica` to fetch the available quantity for each type. The result is stored in the `_availableAlchemica` array.

```bash
function getAvailableAlchemica(uint256 _realmId) public view returns (uint256[4] memory _availableAlchemica) {
    for (uint256 i; i < 4; i++) {
      _availableAlchemica[i] = LibAlchemica.getAvailableAlchemica(_realmId, i);
    }
}
```

#### 2.16 calculateTransferAmounts()
This internal pure function calculates the split of a specified _amount between the owner and the spillover. The spillover is a portion of the total amount that goes to some other address or pool.

`bp` is a constant that represents the "basis points" (10000), so the `_spilloverRate` is multiplied by `10^16` to scale it.

The function calculates the amount that will go to the owner (owner) and the amount that will be spilled (`spill`) based on the given `_spilloverRate`.

It returns a `TransferAmounts` struct containing these two values.


```bash
function calculateTransferAmounts(uint256 _amount, uint256 _spilloverRate) internal pure returns (TransferAmounts memory) {
    uint256 owner = (_amount * (bp - (_spilloverRate * 10 ** 16))) / bp;
    uint256 spill = (_amount * (_spilloverRate * 10 ** 16)) / bp;
    return TransferAmounts(owner, spill);
}
```

#### 2.17 lastClaimedAlchemica()
This is a simple **view** function that returns the last claimed alchemica amount for a given parcel (`_realmId`).

It fetches the value stored in `s.lastClaimedAlchemica` for the given parcel ID.

```bash
function lastClaimedAlchemica(uint256 _realmId) external view returns (uint256) {
    return s.lastClaimedAlchemica[_realmId];
}
```

#### 2.18 claimAvailableAlchemica()
This function allows the **parcel owner** to claim available alchemica for their parcel using a **parent NFT** (likely an Aavegotchi) for verification.

First, it validates the provided `_signature` using `LibSignature.isValid` to ensure the request is legitimate.

Then, it verifies if the caller has access to the parcel with `LibRealm.verifyAccessRight` (using the provided `_gotchiId`).

Finally, it invokes `LibAlchemica.claimAvailableAlchemica` to handle the alchemica claiming process.

``` bash
function claimAllAvailableAlchemica(uint256[] memory _realmIds, uint256 _gotchiId, bytes memory _signature) external gameActive {
    // Check signature
    require(
      LibSignature.isValid(keccak256(abi.encode(_realmIds[0], _gotchiId, s.lastClaimedAlchemica[_realmIds[0]])), _signature, s.backendPubKey),
      "AlchemicaFacet: Invalid signature"
    );

    for (uint256 i; i < _realmIds.length; i++) {
      // 1 - Empty Reservoir Access Right
      LibRealm.verifyAccessRight(_realmIds[i], _gotchiId, 1, LibMeta.msgSender());
      LibAlchemica.claimAvailableAlchemica(_realmIds[i], _gotchiId);
    }
}
```

#### 2.19 getHarvestRates()
 This view function returns the harvest rates for a specific parcel (`_realmId`).

It initializes a new array harvestRates of length 4.

Then, it fetches the alchemica harvest rates from the state variable `s.parcels[_realmId ].alchemicaHarvestRate` for the specified parcel.

It returns the `harvestRates` array, which contains the rates for the 4 types of alchemica.

``` bash
function getHarvestRates(uint256 _realmId) external view returns (uint256[] memory harvestRates) {
    harvestRates = new uint256 ;
    for (uint256 i; i < 4; i++) {
      harvestRates[i] = s.parcels[_realmId].alchemicaHarvestRate[i];
    }
}
```

#### 2.20 getCapacities()
This function returns an array of capacities for each alchemica type for the specified parcel by calling `LibAlchemica.calculateTotalCapacity` for each type.

```bash
function getCapacities(uint256 _realmId) external view returns (uint256[] memory capacities) {
  capacities = new uint256;
  for (uint256 i; i < 4; i++) {
    capacities[i] = LibAlchemica.calculateTotalCapacity(_realmId, i);
  }
}
```

#### 2.21 getTotalClaimed()
This function returns an array showing the total claimed alchemica for each of the 4 types from a given parcel, using LibAlchemica.getTotalClaimed in a loop.

```bash
function getTotalClaimed(uint256 _realmId) external view returns (uint256[] memory totalClaimed) {
  totalClaimed = new uint256;
  for (uint256 i; i < 4; i++) {
    totalClaimed[i] = LibAlchemica.getTotalClaimed(_realmId, i);
  }
}
```
#### 2.22  channelAlchemica()
This function channels alchemica from a parcel to an Aavegotchi after checking signatures, access rights, and verifying altar level and channeling limits, then emits the `ChannelAlchemica` event.


``` bash
function channelAlchemica(uint256 _realmId, uint256 _gotchiId, uint256 _lastChanneled, bytes memory _signature) external gameActive {
  // (validations, kinship calculation, minting/transferring logic, and event emission)
}
```

#### 2.23 getCapacities()
This function returns an array of total capacities for all four types of alchemica (`FUD`, `FOMO`, `ALPHA`, `KEK`) for a given parcel (`_realmId`), using `LibAlchemica.calculateTotalCapacity`.

````bash
function getCapacities(uint256 _realmId) external view returns (uint256[] memory capacities)
````

#### 2.24  getTotalClaimed()
This returns the total claimed amount of each alchemica type for a given parcel.

```bash
function getTotalClaimed(uint256 _realmId) external view returns (uint256[] memory totalClaimed) {
  totalClaimed = new uint256 ;
  for (uint256 i; i < 4; i++) {
    totalClaimed[i] = LibAlchemica.getTotalClaimed(_realmId, i);
  }
}
```

#### 2.25 getLastChanneled()
This is a **read-only (view)** function that returns the **last time** a specific Gotchi channeled alchemica.

In the Aavegotchi realm, "channeling" refers to the action where a Gotchi channels alchemica from a parcel — kind of like spiritually extracting magic tokens from the land.

```bash
function getLastChanneled(uint256 _gotchiId) public view returns (uint256) {
  return s.gotchiChannelings[_gotchiId];
}
```

#### 2.26 getParcelLastChanneled()
This function returns the last time a **specific parcel channeled alchemica**, represented as a Unix timestamp (`uint256`).

It’s a read-only (``view``) function, so it doesn’t cost gas unless called from another on-chain contract.

What is `s.parcelChannelings`?
It’s a mapping that stores the last channeling timestamp for each parcel:

```bash
mapping(uint256 => uint256) parcelChannelings;
```
### Meaning

**Key**: `_parcelId` (the ID of the land/parcel)

**Value**: the last time this parcel was used to channel alchemica

This is like a cooldown clock for each parcel.

_In the Aavegotchi realm, channeling alchemica requires a cooldown period per parcel. The duration of that cooldown depends on the level of the Altar installed on the parcel._

So this function is used to:

- **Enforce cooldowns:**
Before allowing channeling, the smart contract checks this timestamp and makes sure enough time has passed.

- **Enable frontend UIs:**
The frontend (dashboard, dApp UI, etc.) can call this function to display timers or availability.

- **Validate user actions:**
Prevent players from abusing the channeling mechanic — the smart contract uses this internally in `channelAlchemica()`.

### 🏗 Used in
`channelAlchemica()`

Before channeling, the contract checks:

``` bash
require(block.timestamp >= s.parcelChannelings[_realmId] + s.channelingLimits[altarLevel], "Parcel can't channel yet");
```
*_So this function is part of the core anti-spam and game mechanics for the Alchemica system._*

#### 2.27  batchTransferAlchemica()
**🧠 What does it do?**

This Function lets the caller batch transfer all four types of Alchemica (FUD, FOMO, ALPHA, KEK) to multiple recipients in one transaction. This saves on gas and makes it easier for a player, guild, or system to distribute rewards or payments.

**🔢 Parameters**

✅ _targets Type: address[]

Meaning: An array of addresses that will receive the Alchemica.

✅ _amounts Type: uint256[4][]

**Meaning:** A nested array. Each inner array contains 4 values, representing the amount of each alchemica type (FUD, FOMO, ALPHA, KEK) to send to the matching _targets[i].

**Example:**

```bash
_targets = [addr1, addr2]
_amounts = [
  [10e18, 5e18, 2e18, 1e18], // for addr1
  [7e18, 3e18, 0, 0.5e18]    // for addr2
]
```
**🔍 Step-by-Step Explanation**

**🛡 Validation**

```bash
require(_targets.length == _amounts.length, "AlchemicaFacet: Mismatched array lengths");
```
- Makes sure that for each target, there's a corresponding array of 4 token amounts.
- If not, the function reverts to prevent mismatched logic.
  

**🧱 Prepare Tokens**

```bash
IERC20Mintable[4] memory alchemicas = [...];
```
- Prepares references to the four Alchemica ERC20 tokens.

- Uses s.alchemicaAddresses to fetch their contract addresses.
















