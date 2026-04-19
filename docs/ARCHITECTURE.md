# PaCiFaX Architecture

> Status: skeleton. Sections marked TODO need fleshing out as research progresses.

This document is the single source of truth for PaCiFaX's intended hardware implementation. It evolves as we learn. Assume everything is tentative until a subsystem is implemented and verified.

---

## 1. PC-FX system overview

The NEC PC-FX (1994) is a 32-bit CD-ROM console descended from the PC Engine lineage. Key facts:

- **CPU**: NEC V810 @ 21.47727 MHz (32-bit RISC, similar to Virtual Boy's)
- **RAM**: 2 MB main + 256 KB backup + 512 KB video + 256 KB audio
- **Video**: Custom **KING** chip + two HuC6270A-style VDCs + HuC6261-style VCE, YUV-native output with MotionJPEG hardware decode, alpha blending, rotation/scaling
- **Audio**: 6-channel ADPCM + CD-DA passthrough
- **Storage**: Double-speed CD-ROM drive
- **Display**: 256x224 to 352x240 typical, NTSC

## 2. Target platform

- DE10-Nano (Cyclone V SE 5CSEBA6U23I7)
  - ~110K logic elements
  - 5,570 Kb embedded M10K block RAM
  - 112 variable-precision DSP blocks
  - Dual-core ARM Cortex-A9 HPS @ 800 MHz (handles file I/O, menus, glue)
- 128 MB SDRAM module (standard MiSTer add-on)
- MiSTer framework (`sys/` submodule) for video output, input, HPS bridge, OSD

## 3. Top-level block diagram

TODO: ASCII diagram of top module showing V810 ↔ bus arbiter ↔ (KING, VDC-A, VDC-B, VCE, ADPCM, CD iface, main RAM, BIOS ROM). Include clock domains and SDRAM mapping.

## 4. Clock domains

| Domain | Frequency | Consumers |
|---|---|---|
| `clk_sys` | ~85.9 MHz (4x V810) | Core logic, fits 4-phase pipeline |
| `clk_cpu` | 21.47727 MHz | V810 (derived from clk_sys) |
| `clk_vid` | 14.31818 MHz | NTSC pixel clock, VCE output |
| `clk_cd` | TBD | CD-ROM subsystem |

Clock relationships and CDC strategy: TODO.

## 5. Memory map

TODO: full V810 physical address map — ROM, RAM, MMIO windows for KING, VDCs, VCE, ADPCM, CD controller.

## 6. Subsystems

### 6.1 V810 CPU core

- 32-bit RISC, 32 general-purpose registers, 5-stage pipeline
- Hardware floating-point unit (FPU)
- Bit-string operations
- Interrupt controller integrated

**Decision pending**: reuse Virtual Boy core's V810 (if license-compatible and extractable), port an existing open-source V810 (e.g., from Mednafen as behavioral reference only — not HDL), or write from scratch.

Estimated LE budget: 8–12K.

### 6.2 KING (custom chip)

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

### 6.3 VDC-A / VDC-B (HuC6270A-equivalent)

Inherits directly from PC Engine lineage. Two instances run in parallel for background/sprite layering. Substantially similar to existing MiSTer PC Engine core — reuse opportunity likely.

Estimated LE budget: 6–8K each.

### 6.4 VCE (HuC6261-equivalent)

Video output encoder. Mixes VDC outputs with KING's background/MotionJPEG layers, produces final YUV→RGB output for MiSTer's scaler. Priority and chroma-key logic here.

Estimated LE budget: 4–6K.

### 6.5 ADPCM audio

6-channel Okawa-style ADPCM. Sourced from 256 KB audio RAM. Mix with CD-DA stream for final output.

Estimated LE budget: 3–5K.

### 6.6 CD-ROM subsystem

Virtual drive backed by CD image (CUE/BIN, CHD) from SD card via HPS. Handles SCSI-like command protocol, sector buffering, CD-DA audio streaming. Model after existing MiSTer PSX/Saturn/PCE-CD CD subsystems.

Estimated LE budget: 4–6K.

## 7. LE budget summary (preliminary)

| Subsystem | Estimated LEs |
|---|---|
| V810 CPU | 8–12K |
| KING | 30–50K |
| VDC-A + VDC-B | 12–16K |
| VCE | 4–6K |
| ADPCM | 3–5K |
| CD subsystem | 4–6K |
| MiSTer framework + glue | 10–15K |
| **Total estimate** | **70–110K** |

This is tight for Cyclone V's 110K. Optimization and careful sharing (especially in KING) will be required. Validate early via synthesis-only builds of the hardest subsystem.

## 8. Open questions

- Is the existing MiSTer Virtual Boy V810 reusable, and under what license?
- What is the exact MotionJPEG specification used by PC-FX? Standard baseline, or custom variant?
- Can YUV processing be done in YUV end-to-end, or must we convert to RGB internally and back?
- Real PC-FX hardware access: who in the community has a capture setup for cross-validation?
- Does any existing FPGA JPEG decoder (open-source) fit our LE budget?

## 9. Non-goals

- FX-SCSI peripherals (keyboard, mouse)
- PC-FXGA (expansion card for PCs) compatibility
- Backup RAM cart emulation (use internal backup RAM only, at least v1)
- Save states (post-1.0 stretch)
