# Govee H6088 RGBIC Cube Wall Sconces — BLE segment protocol

## Identity

| | |
|---|---|
| Model | H6088 RGBIC Cube Wall Sconces (6 cubes) |
| ADV name pattern | `Govee_H6088_*` |
| Device id format | 8-byte Govee id; BLE MAC is typically the last 6 bytes |
| GATT write UUID | `00010203-0405-0607-0809-0a0b0c0d2b11` (same family as Hexa) |
| Encryption | none observed |

## Cube numbering

Number along the daisy chain for implementers. For a wall map, pick a stable convention (e.g. **1 = left when facing the desk / map left**) and stick to it.

| Cube (chain index) | Mask bit (color packet **byte 12**) |
|---:|---|
| 1 | `0x01` |
| 2 | `0x02` |
| 3 | `0x04` |
| 4 | `0x08` |
| 5 | `0x10` |
| 6 | `0x20` |

Probed 2026-08-04 by painting six distinct colors and reading physical order.

## Packet shape

Every control write is **20 bytes**. **Byte 19 = XOR of bytes 0..18.**

### Cube color (≠ Hexa mask placement)

```text
33 05 15 01 RR GG BB 00 00 00 00 00 MM 00 00 00 00 00 00 XX
MM = LE segment bit (0x01 … 0x20) at byte index 12
```

**Hexa** puts the mask at bytes **9–10**. **H6088** uses **byte 12** only. Same opcode family, different layout.

### Cube brightness

```text
33 05 15 02 LL MM …
```

(`LL` at byte 4; mask bit at byte 5 — same family as H6046-style LE masks.)

### Whole-device

| Action | Prefix |
|---|---|
| Power | `33 01 01` / `33 01 00` |
| Brightness | `33 04 LL` |

## Sample: cube 1 red

```text
33 05 15 01 ff 00 00 00 00 00 00 00 01 00 00 00 00 00 00 dc
```

## Sample: cube 6 cyan

```text
33 05 15 01 00 ff ff 00 00 00 00 00 20 00 00 00 00 00 00 02
```

## How this was found without guessing forever

1. Same GATT UUID as Hexa → try `33 05 15 01` color frame.
2. Hexa-style mask at bytes 9–10 did not map cubes cleanly.
3. H6046-style LE bitmask at **byte 12** mapped 1:1 to six cubes (`0x01`…`0x20`).
4. Live rainbow paint confirmed order.

You can re-verify with a phone app + nRF sniffer, but the tables above are enough to implement.
