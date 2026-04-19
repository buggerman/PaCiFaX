# PaCiFaX Architecture

> Status: skeleton. Sections marked TODO need fleshing out as research progresses.

This document is the single source of truth for PaCiFaX's intended hardware implementation. It evolves as we learn. Assume everything is tentative until a subsystem is implemented and verified.

---

## 1. PC-FX system overview

The NEC PC-FX (1994) is a 32-bit CD-ROM console descended from the PC Engine lineage. Key facts:

- **CPU**: NEC V810 @ 21.47727 MHz (32-bit RISC)
- **RAM**: 2 MB main + 256 KB backup + 512 KB video + 256 KB audio
- **Video**: Custom **KING** chip + two HuC6270A-style VDCs + HuC6261-style VCE, YUV-native output with MotionJPEG hardware decode, alpha blending, rotation/scaling
- **Audio**: 6-channel ADPCM + CD-DA passthrough
- **Storage**: Double-speed CD-ROM drive
- **Display**: 256x224 to 352x240 typical, NTSC

## 2. Project structure

PaCiFaX is split into two cooperating projects:

- **[`v810-hdl`](https://github.com/buggerman/v810-hdl)** — standalone, reusable V810 CPU core in SystemVerilog. Developed and tested independently. Unlocks PC-FX, Virtual Boy, and V810-family arcade boards.
- **`PaCiFaX`** (this repo) — PC-FX-specific subsystems (KING, VDCs, VCE, ADPCM, CD). Consumes `v810-hdl` as a dependency.

This split lets the CPU receive community review independently and remains useful even if PaCiFaX stalls.

## 3. Target platform

- DE10-Nano (Cyclone V SE 5CSEBA6U23I7)
  - ~110K logic elements
  - 5,570 Kb embedded M10K block RAM
  - 112 variable-precision DSP blocks
  - Dual-core ARM Cortex-A9 HPS @ 800 MHz (handles file I/O, menus, glue)
- 128 MB SDRAM module (standard MiSTer add-on)
- MiSTer framework (`sys/` submodule) for video output, input, HPS bridge, OSD

## 4. Top-level block diagram

TODO: ASCII diagram of top module showing V810 ↔ bus arbiter ↔ (KING, VDC-A, VDC-B, VCE, ADPCM, CD iface, main RAM, BIOS ROM). Include clock domains and SDRAM mapping.

## 5. Clock domains

| Domain | Frequency | Consumers |
|---|---|---|
| `clk_sys` | ~85.9 MHz (4x V810) | Core logic, fits 4-phase pipeline |
| `clk_cpu` | 21.47727 MHz | V810 (derived from clk_sys) |
| `clk_vid` | 14.31818 MHz | NTSC pixel clock, VCE output |
| `clk_cd` | TBD | CD-ROM subsystem |

Clock relationships and CDC strategy: TODO.

## 6. Memory map

TODO: full V810 physical address map — ROM, RAM, MMIO windows for KING, VDCs, VCE, ADPCM, CD controller.

## 7. Subsystems

### 7.1 V810 CPU core

**Delivered via [`v810-hdl`](https://github.com/buggerman/v810-hdl).** See that project for CPU-specific architecture, test strategy, and status.

**Status of prior-art reuse**: investigated and ruled out. See [`docs/RESEARCH-V810.md`](RESEARCH-V810.md) for full notes. Key findings:

- No MiSTer Virtual Boy core exists (despite community expectations).
- No open-source V810 HDL implementation exists anywhere, public or academic.
- MAME (C++) and Mednafen (C++) V810 emulators exist — usable as behavioral spec, not as HDL source.
- Writing from scratch is the only path. Estimated 12–18 months for a verified standalone V810.

Estimated LE budget: 8–12K (consumed at PaCiFaX integration time).

### 7.2 KING (custom chip)

The hardest subsystem. Responsibilities:

- System bus arbitration
- DMA controllers
- Background layer with scaling/rotation
- YUV colorspace pipeline
- MotionJPEG decoder
- Chroma keying and alpha blending
- Priority composition with VDCs
- CD-ROM interface glue

MotionJPEG decode is unusual for FPGA retro work. Expect this to consume significant DSP blocks and block RAM for DCT/IDCT and Huffman tables. Study prior FPGA JPEG decoders before committing to an architecture.

Estimated LE budget: 30–50K (largest single subsystem).

### 7.3 VDC-A / VDC-B (HuC6270A-equivalent)

Inherits directly from PC Engine lineage. Two instances run in parallel for background/sprite layering. Substantially similar to existing MiSTer PC Engine core — reuse opportunity likely.

Estimated LE budget: 6–8K each.

### 7.4 VCE (HuC6261-equivalent)

Video output encoder. Mixes VDC outputs with KING's background/MotionJPEG layers, produces final YUV→RGB output for MiSTer's scaler. Priority and chroma-key logic here.

Estimated LE budget: 4–6K.

### 7.5 ADPCM audio

6-channel ADPCM. Sourced from 256 KB audio RAM. Mix with CD-DA stream for final output.

Estimated LE budget: 3–5K.

### 7.6 CD-ROM subsystem

Virtual drive backed by CD image (CUE/BIN, CHD) from SD card via HPS. Handles SCSI-like command protocol, sector buffering, CD-DA audio streaming. Model after existing MiSTer PSX/Saturn/PCE-CD CD subsystems.

Estimated LE budget: 4–6K.

## 8. LE budget summary (preliminary)

| Subsystem | Estimated LEs |
|---|---|
| V810 CPU (from v810-hdl) | 8–12K |
| KING | 30–50K |
| VDC-A + VDC-B | 12–16K |
| VCE | 4–6K |
| ADPCM | 3–5K |
| CD subsystem | 4–6K |
| MiSTer framework + glue | 10–15K |
| **Total estimate** | **70–110K** |

This is tight for Cyclone V's 110K. Optimization and careful sharing (especially in KING) will be required. Validate early via synthesis-only builds of the hardest subsystem.

## 9. Open questions

- What is the exact MotionJPEG specification used by PC-FX? Standard JPEG baseline, or custom variant?
- Can YUV processing be done in YUV end-to-end, or must we convert to RGB internally and back?
- Real PC-FX hardware access: who in the community has a capture setup for cross-validation?
- Does any existing open-source FPGA JPEG decoder fit our LE budget?
- Which MiSTer PC Engine core is the cleanest source for VDC reuse?

## 10. Non-goals

- FX-SCSI peripherals (keyboard, mouse)
- PC-FXGA (expansion card for PCs) compatibility
- Backup RAM cart emulation (use internal backup RAM only, at least v1)
- Save states (post-1.0 stretch)
