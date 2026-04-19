# PaCiFaX Roadmap

> Realistic target: **2–4 years to first playable release** for a dedicated solo developer with prior HDL experience. The V810 is longer than originally estimated — see Phase 1.

PaCiFaX is split into two projects. The CPU is delivered via the standalone [`v810-hdl`](https://github.com/buggerman/v810-hdl) project. This roadmap covers both.

## Phase 0 — Research & Architecture (current)

**Goal**: know exactly what we're building before writing Verilog.

- [x] Investigate reuse of existing MiSTer Virtual Boy core V810 — **ruled out (core does not exist)**
- [x] Survey open-source V810 HDL implementations — **none found**
- [x] Draft V810 reference research note ([`docs/RESEARCH-V810.md`](RESEARCH-V810.md))
- [ ] Mirror Mednafen PC-FX source locally, annotate architecture
- [ ] Complete `docs/ARCHITECTURE.md` — fill all TODO sections (block diagram, memory map, clock domain details)
- [ ] Identify reference hardware source (capture-enabled PC-FX owner in community)
- [ ] Inventory open-source FPGA JPEG decoders; pick reference or decide to write from scratch
- [ ] Announce intent on MiSTer forum and Discord; recruit collaborators

**Exit criteria**: architecture doc is complete and reviewed by at least one experienced MiSTer core developer.

## Phase 1 — V810 CPU (delivered by [`v810-hdl`](https://github.com/buggerman/v810-hdl))

**Goal**: a verified, reusable V810 CPU core in SystemVerilog. Released as its own artifact.

This phase is the longest single phase of the project — **estimated 12–18 months** — because no prior HDL exists and V810 is non-trivial (32 registers, 5-stage pipeline with hardware interlocks, hardware FPU, bit-string ops).

See [`v810-hdl` roadmap](https://github.com/buggerman/v810-hdl) for phase-internal milestones.

**Exit criteria**: passes a suite of C-language test programs compiled via `v810-gcc`, with bit-exact results matching MAME's V810 emulator.

## Phase 2 — Minimum viable system bring-up

**Goal**: boot the PC-FX BIOS to its menu.

- [ ] MiSTer framework integration (`sys/` submodule, video output, HPS glue)
- [ ] Main RAM + SDRAM controller integration
- [ ] BIOS ROM loader via HPS
- [ ] Minimal KING stubs (just enough bus plumbing)
- [ ] Minimal VDC + VCE (static framebuffer output)
- [ ] Get the BIOS animation on screen

**Exit criteria**: PC-FX BIOS boot animation displays correctly.

## Phase 3 — CD-ROM subsystem

- [ ] CD image parser (CUE/BIN, later CHD)
- [ ] SCSI command emulation
- [ ] CD-DA streaming
- [ ] Game disc boot-to-title-screen for at least one game

## Phase 4 — KING & video pipeline

The multi-year slog.

- [ ] Full KING MMIO register map
- [ ] YUV pipeline
- [ ] Background layer with scaling/rotation
- [ ] Chroma keying and alpha blending
- [ ] Priority composition
- [ ] **MotionJPEG decoder** (the hardest single component)

## Phase 5 — Audio

- [ ] ADPCM channels
- [ ] CD-DA mixing
- [ ] Final audio output path

## Phase 6 — Compatibility push

- [ ] Run top 10 commercially-released games, fix regressions
- [ ] Expand to full commercial library
- [ ] Timing accuracy pass against real hardware captures

## Phase 7 — Release 1.0

- [ ] Submit to MiSTer-devel distribution channels
- [ ] Publish release builds
- [ ] Documentation for end users

## Post-1.0

- Save states
- Rewind
- Enhancement modes (if any make sense)
- Backup RAM cart support
- Cross-validation with more community capture hardware
