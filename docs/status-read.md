# Govee BLE status *read* (`aa` queries)

Writes are `33 …`. Status is a **query → notify**: write `aa {cmd} …` on the same GATT write char, read the matching `aa {cmd} …` notify on `…2b10`.

Subscribe to notify (`…2b10` CCCD `01 00`) **before** querying. GATT **read** of `2b10` / `2b11` is not state (`2b10` is often `01 02` or empty).

## Frame

20 bytes. Byte 19 = XOR of 0..18. Same checksum as writes.

```text
aa CC …payload… XX
```

H6061 / H6088: plaintext.  
H600B: encrypt both query and notify with the session key (AES-ECB 16 + RC4 4) after `e701`/`e702` (PSK `MakingLifeSmarte`).

Keep reading notifies until the **decrypted** opcode matches the query. Leftover auth frames will otherwise decode as garbage.

## Whole-device (all three SKUs)

| Field | Query | Reply | Notes |
|---|---|---|---|
| Power | `aa 01` | `aa 01 {0\|1}` | |
| Live brightness % | `aa 04` | `aa 04 {0–100}` | **Not** `aa 12` (`aa 12` is a capability blob, often `ff 64 …`) |
| Mode / solid RGB | `aa 05` | H600B: `aa 05 0d RR GG BB`; Hexa/Sconce: `aa 05 15` + zeros | `0x15` = segment mode, **not** live panel RGB |
| Version | `aa 06`, `aa 20`, `aa 21` | ASCII | |

**Do not** write `33 01 00` / `33 04 00` as a “query” — those are **commands** (power off / dim 0).

## Segment dump (H6061 / H6088 only)

Query **`aa a5 {page}`**. Reply is the same page, four slots:

```text
write:  aa a5 NN 00 … XX
reply:  aa a5 NN  {LL RR GG BB}×4  XX
NN = 01, 02, 03
```

- Page **0** aliases page 1 on Hexa and is zeros on H6088 — skip it.
- Hexa: pages 1–3 = 10 panels + two zero pads (power-first order, same as write masks).
- H6088: pages 1–2 = 6 cubes + pads.
- `LL` is per-segment brightness 0–100. Whole-wall dim is the separate `aa 04` value (colors are often sent at V=100).

Empty `aa 05` does **not** return this dump. Isolated `aa 04` returns only dim. You must query `aa a5` yourself.

### Captured Hexa pages (mint wall, all `LL=100`)

```text
aa a5 01 64 00 ff ae 64 00 ff b7 64 21 ff 97 64 40 ff af 4e
aa a5 02 64 a1 ff e0 64 a1 ff e0 64 40 ff af 64 21 ff 97 54
aa a5 03 64 00 ff b7 64 00 ff ae 00 00 00 00 00 00 00 00 15
```

## H600B

Solid A19/E12. **No segments.** `aa a5` replies are zeros. Read each bulb as its own peripheral (`aa 01` + `aa 04` + `aa 05`).

## Proven 2026-08

Phone open-device sniff + ThinkPad recreate (`aa 04` / `aa a5` isolated). Earlier brute-force suites missed this because they started at `aa 05` and kept only the first notify.
