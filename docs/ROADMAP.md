# PaCiFaX Roadmap

> Realistic target: 2–4 years to first playable release for a dedicated solo developer with prior HDL experience. Adjust expectations accordingly.

## Phase 0 — Research & Architecture (current)

**Goal**: know exactly what we're building before writing Verilog.

- [ ] Mirror Mednafen PC-FX source locally, annotate architecture
- [ ] Verify whether MiSTer Virtual Boy V810 is reusable (license, code quality, extractability)
- [ ] Complete `docs/ARCHITECTURE.md` — fill all TODO sections
- [ ] Finalize top-level block diagram and memory map
- [ ] Identify reference hardware source (capture-enabled PC-FX owner in community)
- [ ] Inventory open-source FPGA JPEG decoders; pick reference or decide to write from scratch
- [ ] Announce intent on MiSTer forum and Discord; recruit collaborators

**Exit criteria**: architecture doc is complete and reviewed by at least one experienced MiSTer core developer.

## Phase 1 — V810 CPU (standalone)

**Goal**: a working V810 core with testbench, independent of PC-FX system integration.

- [ ] Decide: reuse, port, or write from scratch
- [ ] Implement or integrate V810
- [ ] Build a testbench harness (Verilator or iverilog)
- [ ] Pass known V810 test ROMs (reuse Virtual Boy test suites where applicable)
- [ ] Synthesize standalone — verify LE usage matches estimate

**Exit criteria**: V810 passes all available test ROMs bit-exact against Mednafen reference traces.

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
