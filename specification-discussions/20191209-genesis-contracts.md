# Mosaic Implementation Discussion

| version | Last updated | Component          |
| ------- | ------------ | ------------------ |
| 0.14    | 9/12/2019    | Genesis contracts  |

Discussion date/time:

Editor: Benjamin Bollen

Attendees:

## Mosaic contract layout

For Mosaic system contracts we reserve the following addresses, where `0x4d` is chosen for `M`.

```
0x00..000000
0x00..004d**
```

In the `GenesisFile` Mosaic initialises the following contracts
```
0x00..000000 utMOST [proxy]

0x00..004d00 CoConsensus [proxy]
0x00..004d01 CoReputation [proxy]
0x00..004d02 ConsensusCoGateway [proxy]
0x00..004d03 ProtocoreSelf [proxy]
0x00..004d04 OriginObserver [proxy]
0x00..004d05 OriginAnchor [proxy]
...
0x00..004dfa OriginAnchor [mastercopy]
0x00..004dfb OriginObserver [mastercopy]
0x00..004dfc ProtocoreSelf [mastercopy]
0x00..004dfd ConsensusCoGateway [mastercopy]
0x00..004dfe CoReputation [mastercopy]
0x00..004dff CoConsensus [mastercopy]
```

## Genesis initialisation

For each proxy contract in the Mosaic system range, the `GenesisFile` also defines the initial storage of the proxy contract.

As an example we would write for `Protocore`

```js
contract Protocore is MasterCopyUpgradable, GenesisProtocore, ... { }

contract MasterCopyUpgradable {
    /* Storage */
    address mastercopy;
}

contract GenesisProtocore {
    /* Storage */
    bytes32 genesisMetachainId;
    address genesisCore;
    uint256 genesisEpochLength,
    uint256 genesisGasTarget,
    uint256 genesisHeight,
    bytes32 genesisOriginObservation,
    uint256 genesisDynasty,
    uint256 genesisAccumulatedGas,
    bytes32 genesisSource,
    uint256 genesisSourceBlockHeight
}
```
such that as an example the storage for `Protocore` proxy should be written in the `GenesisFile` as
```
slot 00: 00..004dfc
slot 01: <metachaindId>
slot 02: <core>
slot 03: ...
```
All proxies must then be written such that functions can only be called if the genesis parameters have been processed, eg. for `Protocore`

```js
contract Protocore is MasterCopyUpgradable, GenesisProtocore, ... {
    [...]

    function registerVoteMessage() {
        // initialise root vote message from genesis parameters
    }

    function proposeLink(
        bytes32 _sourceVoteMessage,
        ...
    ) {
        // any link must reference a parent, so registerGenesisLink must be called first
    }

    [...]
}

```

## Testing genesis contracts [proposal]

To test such proxy contracts that are initialised through a `GenesisFile`, we can create a `TestGenesisProtocore`

```js
contract TestGenesisProtocore is Protocore {
    // provide constants or constructor to initialise the `genesis*` storage parameters
}
```
