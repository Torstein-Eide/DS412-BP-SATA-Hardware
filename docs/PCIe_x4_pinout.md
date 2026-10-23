# PCIe x4 edge connector — pinout

Only the physical interface (PCIe x4 card edge / socket) is used.
The electrical interface is **NOT** PCIe — the pins have a Synology-specific use.

- **Board:** DS412-BP-CTRLP-SATA
- **Footprint:** `kicad/lib/DS412-BP-SATA.pretty/BUS_PCIexpress_x4.kicad_mod`
  - 32 pins per side (A and B), 64 in total
  - Segment split: 11 + 21 (key between pin 11 and 12)
  - Pitch 1.0 mm
  - Short pins in the footprint: A1 and B31

Signal names are filled in after measurement.

## Pinout

| Pin | Side B (backside of board)         | Side A           | Note                      |
| --: | --------------- | ---------------- | ------------------------- |
|   1 | Ground          | SATA1 present    | A: SATA1 P4               |
|   2 | SATA4 A+        | SATA2 present    | A: SATA2 P4               |
|   3 | SATA4 A-        | Ground           |                           |
|   4 | Ground          | SATA4 B+ via C5  |                           |
|   5 | Ground          | SATA4 B- via C6  |                           |
|   6 | SATA3 A+        | Ground           |                           |
|   7 | SATA3 A-        | Ground           |                           |
|   8 | Ground          | SATA3 B+ via C11 |                           |
|   9 | SATA2 EN        | SATA3 B- via C12 | B: R16 + R9               |
|  10 | SATA3 EN        | Ground           | B: R17 + R8               |
|  11 | SATA4 EN        | SATA1 EN         | B: R18 + R7, A: R15 + R10 |
|     | **KEY / NOTCH** | **KEY / NOTCH**  |                           |
|  12 | Ground          | Ground           |                           |
|  13 | SATA2 A+        | Ground           |                           |
|  14 | SATA2 A-        | Ground           |                           |
|  15 | Ground          | SATA2 B+ via C30 |                           |
|  16 | Ground          | SATA2 B- via C29 |                           |
|  17 | SATA1 A+        | Ground           |                           |
|  18 | SATA1 A-        | Ground           |                           |
|  19 | Ground          | SATA1 B+ via C41 |                           |
|  20 | SATA3 present   | SATA1 B- via C42 | B: SATA3 P4               |
|  21 | SATA4 present   | Ground           | B: SATA4 P4               |
|  22 | +12V            | +5V              | Verified                  |
|  23 | +12V            | +5V              | Verified                  |
|  24 | +12V            | +5V              | Verified                  |
|  25 | +12V            | +5V              | Verified                  |
|  26 | +12V            | +5V              | Verified                  |
|  27 | +12V            | +5V              | Verified                  |
|  28 | +12V            | +5V              | Verified                  |
|  29 | +12V            | +5V              | Verified                  |
|  30 | +12V            | +5V              | Verified                  |
|  31 | +12V            | +5V              | Verified                  |
|  32 | Ground          | Ground           |                           |

## Notes

- **Presence detect (`SATAx_PRES`, A1, A2, B20, B21):** on a SATA connector,
  P4 is a long ground pin, and inside the disk it is tied to the disk's other
  ground pins. The backplane routes P4 to the edge connector instead of to
  ground, so the line is pulled low only when a disk is plugged in.
  - Measured: ~3 V with no disk, 0 V with a disk inserted. Low = disk present,
    3.3 V logic.
  - Still to check: where the pull-up is (original motherboard or backplane).
- **Disk power enable (`SATAx_EN`, A11, B9, B10, B11):**
  - **EN high (~3 V) = disk on. EN low or floating (R10 10k pull-down) =
    disk off.** Disks stay off when nothing drives EN.
  - Measured in the running DS412+: disk running → EN ~3 V. Empty slot →
    EN 0 V and disk +12 V off. The original motherboard only enables
    slots where `SATAx_PRES` is low.
  - Schematic (rev A2): EN → R15 1k → Q101 (assumed NPN, R101 pull-up to
    +5 V) → Q4 base, Q4 collector on EF05/EF12 EN/FAULT. Q101 and R101 are
    not traced; they were added so the drawing gives EN high = disk on. With
    Q4 alone (as first drawn) the polarity was reversed.
  - A short low pulse (~10 ms) on EN resets an e-fuse after a thermal fault.

### TODO

- [x] **Fix the Q4 stage polarity in the schematic** (rev A2, with the
  assumed Q101/R101 inverter).
- [ ] **Trace the real EN stage** to replace the assumed Q101/R101.
  - Read the Q4 marking, find its pinout and trace where C, B and E really
    go. Look for the second transistor (or a PNP) that inverts EN.
- [ ] **Bench check (optional), backplane only, 12 V + 5 V applied, one disk
  inserted:** EN floating → disk +12 V (P13) off; EN to 3.3 V through 1k →
  on. Measure e-fuse pin 3 too: ~2.6–3.3 V = enabled, <0.8 V = disabled,
  ~1.4 V = thermal fault.
- [ ] **Check the ramp time.** C4 (value unknown) is probably on dV/dt (pin 2)
  of one e-fuse.
  - Confirm which one with continuity, then measure C4 or time the rise of
    disk +12 V (P13) / +5 V (P7) with a scope.
  - STEF12: Δt = (C + 35.3 pF) / 3.92×10⁻⁸

    | C (dV/dt) | Ramp time |
    | --------- | --------- |
    | none      | ~1 ms     |
    | 1 nF      | ~26 ms    |
    | 4.7 nF    | ~121 ms   |
    | 10 nF     | ~256 ms   |

  - Check whether the other e-fuse also has a dV/dt cap (maybe C24).
  - In the schematic, change C4 to a non-polarized capacitor.
