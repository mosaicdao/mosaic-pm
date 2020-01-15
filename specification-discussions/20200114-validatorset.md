---
title: 'ValidatorSet'
disqus: https://hackmd.io/qF0_wI58QuyDp2k53ylr9A
---

# ValidatorSet for Core & Protocore

| Version | Last Updated | Component    |
| ------- | ------------ | ------------ |
| 0.14    | 14/01/2020   | ValidatorSet |

**Editor:** Paruyr Gevorgyan

---

## Overview

- Core and Protocore ValidatorSet should be eventually consistent.
- Insert/Remove validators should ignore the fact if validator exists or if it has been already removed.
- As a proposal, we do not need also check if they are slashed or not once calculated the quorum. This will be updated in the next round (logged out).
- upsert(height, rep=0) => logout at height (or do nothing)
- upsert(height, rep > 0) => do nothing if "known" or join at height


## User Stories

- As coconsensus I should be able to add a validator.
- As coconsensus I should be able to remove a validator.
- As a user I should be able to ask from the contract if a validator is
  in set for a given metablock height.
- As a user I should be able to ask from the contract if a validator is
  in the forward set for a given metablock height.
- As a user I should be able to ask from the contract if a validator is
  in the rear set for a given metablock height.

## Proposed Implementation

```solidity

contract ValidatorSet {

    /** Maximum future end height, set for all active validators */
    uint256 public constant MAX_FUTURE_END_HEIGHT = uint256(
        0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff);

    /**
     * Validator begin height assigned to this set:
     *   - zero: not registered to this set, or started at height 0
     *           if endHeight > 0
     *   - bigger than zero: begin height for active validators
     *                       (given endHeight >= metablock height)
     */
    mapping(address => uint256) public validatorBeginHeight;

    /**
     * Validator end height assigned to this set:
     *   - zero: not registered to this set
     *   - MAX_FUTURE_END_HEIGHT: for active validators (assert beginHeight <= metablock height)
     *   - less than MAX_FUTURE_END_HEIGHT: for logged out validators
     */
    mapping(address => uint256) public validatorEndHeight;

    /**
     * @notice Inserts validator in the set.
     *         Sets begin height and end height (to the max future end height)
     *         of a validator.
     */
    function insertValidator(address _validator, uint256 _beginHeight)
        internal
    {
        // ...
    }

    /**
     * @notice Removes validator from the set.
     */
    function removeValidator(address _validator, uint256 _endHeight)
        internal
    {
        // ...
    }

    function inValidatorSet(address _validator, uint256 _height)
        public
        view
        returns (bool)
    {
        return validatorBeginHeight[_validator] <= _height &&
            validatorEndHeight[_validator] >= _height &&
            validatorEndHeight[_validator] > 0;
    }

    function inForwardValidatorSet(address _validator, uint256 _height)
        public
        view
        returns (bool)
    {
        return validatorBeginHeight[_validator] <= _height &&
            validatorEndHeight[_validator] > _height &&
            validatorEndHeight[_validator] > 0;
    }
}

```