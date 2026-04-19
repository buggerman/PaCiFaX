# PaCiFaX

**NEC PC-FX core for MiSTer FPGA (DE10-Nano).**

> Status: Pre-alpha. Architecture and research phase. No playable code yet.

PaCiFaX is an open-source project to bring NEC's 1994 PC-FX console to the MiSTer FPGA platform via a cycle-accurate Verilog/SystemVerilog core running on the Intel Cyclone V SoC in the Terasic DE10-Nano.

## Why

The PC-FX is the last major mainstream Japanese 32-bit console without a serious FPGA implementation. Its library — FMV-heavy, anime-adjacent, cult-classic — is underserved by every existing emulation option outside Mednafen. A MiSTer core closes one of the last meaningful gaps in retro hardware preservation achievable on current-generation consumer FPGAs.

## Status

Currently in **Phase 0: Research & Architecture**. See [`docs/ROADMAP.md`](docs/ROADMAP.md) for the phased plan.

## Hardware target

- **Board**: Terasic DE10-Nano (or MiSTer-compatible clones: MiSTer Pi, etc.)
- **FPGA**: Intel Cyclone V SE 5CSEBA6U23I7 (~110K LEs, 5.5 Mb block RAM, 112 DSP blocks)
- **Required add-ons**: 128 MB SDRAM module, USB hub, CD-ROM image support via SD card

## Architecture overview

See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

Major subsystems to implement:

- NEC V810 CPU (32-bit RISC, 21.5 MHz)
- **KING** — custom system controller and video coprocessor (the hard part)
- HuC6270A-equivalent background/sprite unit (VDC, from PC Engine lineage)
- HuC6261-equivalent video output encoder / VCE
- ADPCM audio (CD-DA + streaming)
- CD-ROM interface + BIOS loader
- YUV-native video pipeline with MotionJPEG decode

## Reference material

- Mednafen PC-FX emulator source (GPL-2.0+) — authoritative behavior reference
- Existing MiSTer Virtual Boy core — possible V810 CPU starting point (verify extractability)
- pcfxtra and community technical documentation
- PC-FX BIOS disassembly notes (community-sourced)

## Building

Not yet available.

## Contributing

Early-stage project. If you have HDL experience and want to help with any subsystem (V810 CPU, KING, audio, CD subsystem), open an issue to coordinate. Pull requests welcome once scaffolding stabilizes.

## License

GPL-3.0-or-later. See [`LICENSE`](LICENSE).

This project is not affiliated with NEC, Mednafen, or the MiSTer project. PC-FX is a trademark of its respective owner.
