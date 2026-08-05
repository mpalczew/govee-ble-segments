# Govee H6061 Glide Hexa — BLE segment protocol

## Identity

| | |
|---|---|
| Model | H6061 Glide Hexa (10 panels) |
| ADV name pattern | `Govee_H6061_*` |
| GATT write UUID | `00010203-0405-0607-0809-0a0b0c0d2b11` |
| Encryption | none (nRF sniffer, 2026-07) |

## Panel numbering (power-first)

**1 = power injector** (cable end of the chain). Then along the physical snake to the far end (**10**).

The Govee app does not label indices. This map is for implementers.

| Panel | Mask (u16 BE at color bytes 9–10) |
|---:|---|
| 1 | `0x0100` |
| 2 | `0x0200` |
| 3 | `0x0400` |
| 4 | `0x0800` |
| 5 | `0x1000` |
| 6 | `0x2000` |
| 7 | `0x4000` |
| 8 | `0x8000` |
| 9 | `0x0001` |
| 10 | `0x0002` |

Derived from sniffer + eye checks, then flipped so power is panel 1.

## Packet shape

Every control write is **20 bytes**. **Byte 19 = XOR of bytes 0..18.**

### Segment color

```text
33 05 15 01 RR GG BB 00 00 MH ML 00 00 00 00 00 00 00 00 XX
mask = (MH << 8) | ML
```

### Segment brightness

```text
33 05 15 02 LL MH ML 00 …
```

(`LL` = level 0–100; mask same BE placement as color but at bytes 5–6.)

### Whole-device

| Action | Prefix |
|---|---|
| Power | `33 01 01` / `33 01 00` |
| Brightness | `33 04 LL` |

**Note:** Whole-device brightness dims the wall without changing segment hues. Colors are typically sent at full value; wall dim is separate.

## Sample: panel 1 red

```text
33 05 15 01 ff 00 00 00 00 01 00 00 00 00 00 00 00 00 00 dc
```

## Capture notes (if you re-verify)

- nRF52840 sniffer must see `CONNECT_IND` — start sniffer **before** the phone connects.
- Force-quit Govee when your controller is central.
