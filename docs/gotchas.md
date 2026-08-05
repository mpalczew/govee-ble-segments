# Gotchas (read these before debugging)

## One BLE central

Govee devices allow **one** active central. If the **phone app is connected**, your Linux/Mac controller will fail connects or writes. **Force-quit Govee** on the phone before testing.

## Phone already connected ⇒ empty sniffer PCAPs

nRF / Wireshark sniffer hops onto a connection only if it sees **`CONNECT_IND`**. If the phone was already connected when you start the sniffer, you only get advertisements — no ATT segment traffic.

Order: start sniffer → force-quit app → open app and paint.

## Whole-device vs segment brightness

- `33 04 LL` — whole wall/row dim (does not clear hues).
- `33 05 15 02` — per-segment brightness (can leave the room “dim” even if LAN reports 100%).

If the wall looks stuck dim after experiments, raise **section** levels (app or segment brightness writes), not only whole-device 100%.

## Do not flash wall brightness 100 on every reconnect

Some controllers power-on then force `33 04 64` on connect. That causes a full-bright flash mid-scene. Prefer: power on → paint segments → set wall dim once.

## Adapter contention

If something else is scanning (hygrometers, continuous BLE scan) with exclusive access for seconds at a time, multi-segment paints fragment: first cubes/panels update, rest wait or fail. Hold the adapter for the **entire** multi-write paint.

## LAN color does not replace segment paint cleanly

After a multi-color BLE paint, LAN `colorwc` alone may not fully overwrite all segments. For solid looks, paint **all** segments the same color over BLE (or re-paint via app).

## H6061 vs H6088 mask placement

Same `33 05 15 01` color opcode. **Different mask bytes.** Copying Hexa masks onto H6088 will not work.
