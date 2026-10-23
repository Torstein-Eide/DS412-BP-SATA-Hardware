# DS412-BP-SATA

Reverse-engineered KiCad schematic and pinout of the SATA backplane in the
Synology DS412+ NAS.

- Silkscreen: `Model: DS412-BP-CTRLP-SATA  P/N: 08-0441CP11  VER: 1.1`
- Secondary marking: `S012J147C-2`

The goal is to document the board well enough that a replacement
motherboard can drive it: power the four disks on and off one by one
(staggered spin-up), see which bays hold a disk, and route the four SATA
links.

## What the board does

- Four SATA 22-pin combo connectors (7-pin data + 15-pin power), one per
  drive bay. The drives blind-mate into them.
- E-fuses in MSOP-10 packages switch +12 V and +5 V to the disks. They are
  marked `EF12` and `EF05`; the exact part is not identified, but the pinout
  seems to follow the ST STEF12. A disk is powered only while its
  `SATAx_EN` line is driven high.
- Disk presence detect: SATA power pin P4 (a ground pin inside the disk) is
  routed to the edge connector, so `SATAx_PRES` is pulled low when a disk is
  inserted.
- Connects to the motherboard through a **PCIe x4 card edge** (64 pins,
  1.0 mm pitch, key between pins 11 and 12). Only the mechanical form is
  PCIe. The pins carry Synology-specific signals: four SATA links, the EN and
  PRES lines, +12 V, +5 V and ground.

### Control signals (measured)

| Signal       | Edge pins          | Level                                         |
| ------------ | ------------------ | --------------------------------------------- |
| `SATAx_EN`   | A11, B9, B10, B11  | High (~3 V) = disk on; low or floating = off  |
| `SATAx_PRES` | A1, A2, B20, B21   | ~3 V = empty bay; 0 V = disk inserted         |

A short (~10 ms) low pulse on `EN` resets an e-fuse after a thermal fault.
See [docs/PCIe_x4_pinout.md](docs/PCIe_x4_pinout.md) for the full pinout.

### Mechanical

- The gold edge connector has two segments (11 + 21 pins per side). It plugs
  directly into a black two-segment socket on the original DS412+
  motherboard, near the `P2`/`CONN2` marking. There is no cable.
- Length is roughly 140–155 mm (estimated from photos, not yet measured).
- There are four cutouts between the SATA connectors for the drive mounting.
- Still to measure with calipers: PCB thickness at the edge connector and the
  exact key position.

## Principle drawing

The schematic is a **principle drawing**, not a manufacturing schematic. It
shows how the board works, traced and measured from one board:

- Only the SATA1 channel is drawn. SATA2–4 use the same circuit; their edge
  connector pins are marked no-connect.
- Parts that were not traced are marked with `?`, and parts numbered 100 and
  up (Q101, R101) are assumed, added so the drawing matches the measured
  behaviour.
- There is no PCB layout, and there are no plans for one.

## Repository contents

| Path                                  | Contents                                              |
| ------------------------------------- | ----------------------------------------------------- |
| `kicad/DS412-BP-SATA.kicad_pro` / `_sch` | KiCad project and principle schematic                 |
| `kicad/lib/DS412-BP-SATA.kicad_sym`   | Project symbols: edge connector, SATA connector, e-fuses |
| `kicad/lib/DS412-BP-SATA.pretty/`     | Footprints: PCIe x4 card edge, SATA combo connector   |
| `docs/PCIe_x4_pinout.md`              | Edge connector pinout, measurements, open questions   |
| `docs/SATA_pinout.md`                 | SATA 22-pin connector pinout and board notes          |

Open `kicad/DS412-BP-SATA.kicad_pro` with KiCad 9 or newer. Some passives use the AKL libraries from the
KiCad Plugin and Content Manager (`PCM_Capacitor_AKL`, `PCM_Resistor_AKL`,
`PCM_Generic`); their symbols are embedded in the schematic, so it opens
without them.

## Status

Work in progress. Known open items:

- The `EN` input stage is not traced. The drawing uses an assumed inverter
  (Q101, R101) in front of Q4 to match the measured polarity.
- Location of the `PRES` pull-up (motherboard or backplane) is not confirmed.
- The e-fuse part number, pins 2–5, and the dV/dt capacitor value (C4, ramp
  time) are not confirmed.

Details and TODOs are in [docs/PCIe_x4_pinout.md](docs/PCIe_x4_pinout.md).
Corrections and measurements from other DS412+ owners are welcome.

## Datasheets

Third-party datasheets are not included. Get them from the manufacturer,
for example the [STEF12](https://www.st.com/en/power-management/stef12.html)
from ST.

## License

MIT, see [LICENSE](LICENSE). The two footprints in
`kicad/lib/DS412-BP-SATA.pretty/` are copied from the official KiCad library and
keep their original license (CC-BY-SA 4.0 with the KiCad library
exception).

Synology and DS412+ are trademarks of Synology Inc. This project is not
affiliated with or endorsed by Synology.
