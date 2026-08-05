# Sample 20-byte packets

Checksum: last byte = XOR of the first 19.

## Common

```text
# power on
33 01 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 33

# power off
33 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 32

# whole-device brightness 100 (0x64)
33 04 64 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 53

# whole-device brightness 10 (0x0a)
33 04 0a 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 3d
```

## H6061 Hexa — segment color (mask BE @ bytes 9–10)

```text
# panel 1 red  (mask 0x0100)
33 05 15 01 ff 00 00 00 00 01 00 00 00 00 00 00 00 00 00 dc

# panel 10 cyan (mask 0x0002)
33 05 15 01 00 ff ff 00 00 00 02 00 00 00 00 00 00 00 00 20
```

## H6088 sconces — cube color (mask bit @ byte 12)

```text
# cube 1 red  (0x01)
33 05 15 01 ff 00 00 00 00 00 00 00 01 00 00 00 00 00 00 dc

# cube 6 cyan (0x20)
33 05 15 01 00 ff ff 00 00 00 00 00 20 00 00 00 00 00 00 02
```

## Pseudocode

```python
def xor20(payload19: bytes) -> bytes:
    assert len(payload19) == 19
    x = 0
    for b in payload19:
        x ^= b
    return payload19 + bytes([x])

def hexa_seg_color(mask: int, r: int, g: int, b: int) -> bytes:
    p = bytearray(19)
    p[0:4] = bytes([0x33, 0x05, 0x15, 0x01])
    p[4:7] = bytes([r, g, b])
    p[9] = (mask >> 8) & 0xFF
    p[10] = mask & 0xFF
    return xor20(bytes(p))

def sconce_cube_color(mask_bit: int, r: int, g: int, b: int) -> bytes:
    p = bytearray(19)
    p[0:4] = bytes([0x33, 0x05, 0x15, 0x01])
    p[4:7] = bytes([r, g, b])
    p[12] = mask_bit & 0xFF
    return xor20(bytes(p))
```

Write the 20 bytes to characteristic  
`00010203-0405-0607-0809-0a0b0c0d2b11` (Write Without Response works).
