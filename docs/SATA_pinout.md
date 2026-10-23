# SATA pinout — 22-pin combo connector (7-pin data + 15-pin power)

- **Board:** DS412-BP-CTRLP-SATA (4 connectors, one per disk)
- Standard SATA connector towards the disk (blind-mate).

**Direction:**

- A = host transmits → disk receives
- B = disk transmits → host receives

**Mating order (hot-plug):**

1. First — long pins
2. Then — medium pins (precharge)
3. Last — short pins

## Data (7-pin)

| Pin | Signal | Description              | Mating |
| --- | ------ | ------------------------ | :----: |
| S1  | GND    | Ground                   |   1    |
| S2  | A+     | Diff. pair A + (host TX) |   2    |
| S3  | A-     | Diff. pair A - (host TX) |   2    |
| S4  | GND    | Ground                   |   1    |
| S5  | B-     | Diff. pair B - (host RX) |   2    |
| S6  | B+     | Diff. pair B + (host RX) |   2    |
| S7  | GND    | Ground                   |   1    |

## Power (15-pin)

| Pin | Signal  | Description                                                            | Mating |
| --- | ------- | ---------------------------------------------------------------------- | :----: |
| P1  | +3.3V   | 3.3V (often unused on 3.5" disks)                                      |   3    |
| P2  | +3.3V   | 3.3V                                                                   |   3    |
| P3  | +3.3V   | 3.3V precharge / PWDIS (SATA 3.3+)                                     |   2    |
| P4  | GND     | Ground — on this board: disk presence detect (see `PCIe_x4_pinout.md`) |   1    |
| P5  | GND     | Ground                                                                 |   2    |
| P6  | GND     | Ground                                                                 |   2    |
| P7  | +5V     | 5V precharge                                                           |   2    |
| P8  | +5V     | 5V                                                                     |   3    |
| P9  | +5V     | 5V                                                                     |   3    |
| P10 | GND     | Ground                                                                 |   2    |
| P11 | DAS/DSS | Activity LED / staggered spin-up                                       |   3    |
| P12 | GND     | Ground                                                                 |   1    |
| P13 | +12V    | 12V precharge                                                          |   2    |
| P14 | +12V    | 12V                                                                    |   3    |
| P15 | +12V    | 12V                                                                    |   3    |

## Remarks

- **P3 PWDIS:** on disks with SATA 3.3 PWDIS support, the disk is held off if
  3.3V is present on P3. Check whether the backplane supplies 3.3V.
- **P11:** the disk drives it low = activity (LED). The host can hold it low at
  power-up to disable staggered spin-up.
- **Current per disk** (3.5", typical): 12V ~2 A at spin-up, 5V ~0.7 A.

## Notes

- C25, C33, C34 share a common point to ground.
- R11–R14 are connected to P13.
- SATA P13–P15 are shorted together.
