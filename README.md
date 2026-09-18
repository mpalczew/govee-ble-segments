# Govee BLE segment + status protocol — H6061, H6088, H600B

**Local, per-segment color control over BLE** for two Govee wall products that most open-source stacks only drive as whole-device LAN lights — plus the **status-read** opcodes (`aa 04` dim, `aa a5` segment dump) and the **H600B** solid bulb (encrypted, **no segments**).

If you searched for:

- `Govee H6061 BLE` / `Hexa segment color`
- `Govee H6088` / `cube sconces` individual panels
- `33 05 15` Govee BLE / `00010203-0405-0607-0809-0a0b0c0d2b11`
- Govee segment mask / RGBIC wall BLE

…this is the decoded wire format so you **do not need an nRF sniffer** to paint panels.

| SKU | Product | Segments | Notes |
|-----|---------|----------|--------|
| **H6061** | Glide Hexa (10 hex panels) | 10 | Color mask bytes **9–10** (BE) |
| **H6088** | RGBIC Cube Wall Sconces | 6 cubes | Color mask **byte 12** (LE bit) |
| **H600B** | E12 / A19 RGB bulb | **none** | Encrypted session; solid `33 05 0d` only |

H6061 vs H6088: same GATT family, **different mask layout** — easy to mix up.

## Status

Reverse-engineered from air sniffs + live paint/status tests (2026-07 / 2026-08). Used in production on a private homelab controller. No Govee cloud required.

## Docs

| File | Contents |
|------|----------|
| [docs/h6061-hexa.md](docs/h6061-hexa.md) | Hexa identity, power-first panel map, packets |
| [docs/h6088-sconces.md](docs/h6088-sconces.md) | Cube map, byte-12 masks, packets |
| [docs/h600b.md](docs/h600b.md) | Solid encrypted bulb — not a segment device |
| [docs/status-read.md](docs/status-read.md) | `aa 01` / `aa 04` / `aa 05` / `aa a5` queries |
| [docs/samples.md](docs/samples.md) | Ready-to-write hex dumps |
| [docs/gotchas.md](docs/gotchas.md) | One-central rule, phone app, wall vs segment dim |

## Minimal frame shape (both devices)

Every control write is **20 bytes**. **Byte 19 = XOR of bytes 0..18.**

```text
# power on
33 01 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 33

# whole-device brightness 100%
33 04 64 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 53

# segment color (see device docs for mask bytes)
33 05 15 01 RR GG BB … mask … XX
```

GATT write characteristic (common Govee family):

`00010203-0405-0607-0809-0a0b0c0d2b11`

Encryption: **none** on H6061 / H6088. **H600B is encrypted** (see [h600b.md](docs/h600b.md)).

## Quick start (conceptual)

1. Force-quit the **Govee phone app** (one BLE central at a time).
2. Connect to the ADV name (`Govee_H6061_*` or `Govee_H6088_*`).
3. Write power-on, then segment colors, then whole-device brightness.
4. Hold exclusive access to the adapter for a full multi-segment paint (don’t interleave long scans).

A full paint is ~10 or 6 color writes + one wall dim, with ~80 ms between segment writes.

## Related open source

- [wez/govee2mqtt](https://github.com/wez/govee2mqtt) — excellent LAN / IoT; segment BLE maps like these are still scarce.
- [egold555/Govee-Reverse-Engineering](https://github.com/egold555/Govee-Reverse-Engineering) — community RE hub.
- Strip RE (same opcode family, different products): various H6127 BLE writeups.

## Traffic / discovery

- **Views & clones:** GitHub → **Insights → Traffic** (repo owner; 14-day rolling).
- **Stars / forks** also show up under Insights.

## Hardware

I reverse-engineer Govee lights. Any SKU that still needs work is useful, not only the three above.

- **Device:** mail it (US). Open an [issue](https://github.com/mpalczew/govee-ble-segments/issues) first so I can send a ship-to. I keep the hardware and credit you in the product note.
- **Money:** any channel. I spend it on Amazon for the next SKU.

## License

[MIT](LICENSE) — protocol documentation and sample packets. No Govee trademarks claimed; product names for identification only.

## Disclaimer

Interoperability research on hardware you own. Not affiliated with Govee. Use at your own risk; force-quit their app before connecting or paints will fail / fight.
