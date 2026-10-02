# TagMod

TagMod is a small companion/adapter PCB ("mod") for a SEGGER J-Link debug
probe. It plugs into the J-Link's 20-pin ARM JTAG/SWD header (J101) and
re-fans the signals out to a target-side 10-pin header (J103) plus a
dedicated UART header (J1), and it can supply the target with a
resistor-selectable 1.8 V / 3.3 V rail generated on-board from a USB-C
5 V input.

The earlier working name for this design was `jlink-uart-mod` (see
`output/jlink-uart-mod-a-202503092059.zip`); the project files were
renamed to match this repository in commit `7d403b9`.

## What it is for

Bringing up and debugging small Golioth hardware designs that expose a
compact 10-pin SWD + UART debug connection (e.g. via a Tag-Connect-style
cable — note the `TC TX` / `TC RX` silkscreen) instead of a full 20-pin
ARM header. One board carries SWD, target UART (through the J-Link's
built-in VCOM), and target power between the J-Link and the device under
test. Carries a 5 mm Golioth logo footprint (`footprints.pretty/the_golioth_5mm`).

## Hardware summary

Derived from `TagMod.kicad_sch` / `TagMod.kicad_pcb` (no other docs exist):

| Ref  | Part / footprint | Function |
|------|------------------|----------|
| J101 | 2x10 2.54 mm pin header (Conn_ARM_JTAG_SWD_20) | Input from J-Link 20-pin ARM JTAG/SWD header (SWDIO, SWCLK, RESET_N, TRST, RTCK, TDO/SWO, J-Link UART TX/RX) |
| J103 | 2x05 2.54 mm pin header (Conn_ARM_JTAG_SWD_10) | Target-side SWD output (10-pin ARM Cortex / Tag-Connect-ribbon-compatible) |
| J1   | 1x03 2.54 mm pin header | Target UART breakout (TARGET_TX / TARGET_RX / GND) |
| U4   | GT-USB-7010ASV USB-C connector | 5 V input only (CC1/CC2 terminated with 5.1 kΩ; D+/D-/SBU unconnected) |
| U1   | AMS1117-ADJ (SOT-89) | Adjustable LDO generating the VOUT target rail from the 5 V input |
| D1, D2 | CUS10S30 Schottky (SOD-323) | Rail steering / protection |
| 2x LED (0603, red) | — | `Input Power` and `Output Power` indicators (silkscreen) |
| TP101-103 | 2.0 mm test points | Bench access |
| —    | the_golioth_5mm | Golioth logo silkscreen artwork |

Signal flow (net names): `JLINK_TX`/`JLINK_RX` (J-Link VCOM) are crossed to
`TARGET_TX`/`TARGET_RX`; `SWDIO`, `SWCLK`, `RESET_N` route J101 to J103.
The `ADJ`/`VOUT` divider (6.1 kΩ reference with 2.7 kΩ / 10 kΩ options,
selected with 0-ohm jumpers) gives nominal **1.8 V or 3.3 V** target
supply, matching the `1V8` / `3V3` silkscreen selectors.

## Repository contents

- `TagMod.kicad_pro`, `TagMod.kicad_sch`, `TagMod.kicad_pcb`, `TagMod.kicad_prl` — KiCad 9.0 project (single-sheet schematic, 2-layer PCB)
- `footprints.pretty/the_golioth_5mm.kicad_mod` — local Golioth logo footprint
- `fp-lib-table`, `sym-lib-table` — library tables (note: several symbols/footprints come from an `easyeda2kicad` library that is NOT in this repo; see Known gaps)
- `output/gerber/` — rev B fabrication outputs (gerbers + drill)
- `output/jlink-uart-mod-a-202503092059.zip` — gerber bundle from the earlier rev-A-era `jlink-uart-mod` design (March 2025)
- `TagMod-backups/` — 9 KiCad auto-backup zips (tracked in git; should not be)
- `input/`, `input/3d/` — tracked only via stray `.DS_Store` files; actual contents unknown [UNKNOWN]

## Status (October 2026)

- 4 commits, all 2026-01-21 to 2026-01-23. No activity since.
- Rev A: initial board had the J-Link UART TX/RX flipped (commit `920f7ca`).
- Rev B: TX/RX swapped, renamed to TagMod, "passes DRC, ready for fab"
  (commit `7862415`, 2026-01-22). Gerbers in `output/gerber/` are the
  rev B outputs.
- Whether rev B was ever fabricated, assembled, or electrically validated:
  [UNKNOWN].
- **Known issue — regulator circuit needs further investigation.** Per the
  designer (October 2026), the AMS1117-ADJ target-rail regulator had
  problems; the root cause is not captured in this repo. Before any rev B
  fabrication or field use, validate the LDO section (U1 plus the ADJ/VOUT
  divider — 6.1 kΩ reference with 2.7 kΩ / 10 kΩ 0-ohm select options for
  1.8 V / 3.3 V — and the D1/D2 rail steering) on the bench, loaded and
  unloaded.
- Design tool: KiCad 9.0 (schematic format 20250114, PCB format 20241229).

## Known gaps (pre-open-sourcing checklist)

- **No LICENSE file.** A license (e.g. CERN-OHL, Apache-2.0, MIT) must be
  chosen and added before publishing.
- **Tracked junk:** `.DS_Store` (x3: root, `footprints.pretty/`, `input/3d/`),
  `fp-info-cache` (KiCad cache), `~TagMod.kicad_pcb.lck` (lock file),
  `TagMod-backups/*.zip` (9 auto-backups), and `output/*`. The `.gitignore`
  added in the final commit (`45dc6e3`) already covers `*.zip`,
  `/*-backup/*`, `output/*`, and `*.lck`, but the files were never removed
  from the index (`git rm --cached` needed).
- **No schematic/PCB title-block metadata** (no title, author, revision, or
  date fields set in the KiCad files) — this README is the only description
  of the project.
- **External symbol/footprint dependency:** the design uses an
  `easyeda2kicad` library (AMS1117, USB-C connector, LEDs, diodes, caps)
  that is not vendored into this repo; the schematic will not render
  correctly on a fresh checkout without it.
- **Stale artifact:** `output/jlink-uart-mod-a-202503092059.zip` predates
  this repository (March 2025) and reflects the old project name/rev A.
- **No BOM, assembly drawing, or fabrication notes.**
- Secrets sweep (2026-10): clean. No credentials, keys, or personal emails
  in any tracked file. (The two 24-hex strings in `fp-info-cache` are KiCad
  footprint-cache hashes, and `psk` matches are the Murata "TPSKA"
  resonator part name — both false positives.)
