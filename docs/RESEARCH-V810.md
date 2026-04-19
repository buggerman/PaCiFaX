# Research note: NEC V810 reference landscape

> Compiled to answer the question "What can we reuse or reference to build a V810 HDL core?"
>
> **Bottom line**: nothing in HDL. Everything must be written from scratch. Software emulators and documentation are plentiful and authoritative.

## Key findings

1. **No MiSTer Virtual Boy core exists.** Despite long-standing community discussion, there is no released or in-progress MiSTer core for Virtual Boy as of early 2026. A public tweet dated 2025 is literally titled "Why the MiSTer Still Doesn't Have a Virtual Boy Core."
2. **No open-source V810 HDL implementation exists anywhere.** GitHub, OpenCores, academic releases — all empty. The V810 is genuinely un-implemented in synthesizable hardware outside of NEC's original silicon.
3. **Software emulators are mature and can serve as a behavioral specification.**
4. **Toolchain support exists**, which makes test-ROM-driven development feasible from day one.

## Implication for PaCiFaX

The V810 is now the **critical-path blocker** for the entire project. There is no shortcut. This is why we've split the CPU into a standalone [`v810-hdl`](https://github.com/buggerman/v810-hdl) project — so it can progress independently and deliver value (to Virtual Boy, V810 arcade cores, research projects) even if PaCiFaX itself stalls.

## Reference material inventory

### Primary specification

- **NEC V810 family user manual** — `u10082ej1v0um00.pdf`, archived at virtual-boy.com. The original NEC document. Authoritative for opcode encoding, register behavior, interrupt model, pipeline hazards.
- **NEC uPD70732 datasheet** — electrical and timing specification for the specific silicon variant.

### Behavioral references (C++, for functional spec only)

- **MAME V810 emulator** — `src/devices/cpu/v810/v810.cpp` in [mamedev/mame](https://github.com/mamedev/mame). GPL-2.0. Used widely and well-tested. Can be treated as a reference oracle for instruction-level behavior.
- **Mednafen PC-FX** — its V810 is separate from MAME's and has been hardened against real PC-FX software for years. GPL-2.0+.

> These are **C++ software emulators**. We cannot copy code from them into HDL. We *can* use them as specifications and as reference oracles to compare our HDL against.

### Toolchain and tooling

- **[jbrandwood/v810-gcc](https://github.com/jbrandwood/v810-gcc)** — GCC 4 patches + build scripts producing a working V810 C toolchain. Means we can write test ROMs in C.
- **[20Enderdude20/Ghidra_v810_v830](https://github.com/20Enderdude20/Ghidra_v810_v830)** — Ghidra processor module for disassembling V810 binaries. Useful for reverse engineering BIOS and validating instruction encodings.

### Community documentation

- Virtual Boy Architecture analysis (Rodrigo Copetti, copetti.org) — accessible prose breakdown of the V810 pipeline and its implications.
- Planet Virtual Boy wiki — homebrew-community-maintained V810 notes, errata, and test programs.
- PC-FX technical documentation (community-sourced, scattered) — register maps, MMIO windows, KING/VCE behavior.

## Validation strategy for v810-hdl

Because we have a working C toolchain and two reference emulators, we can do real test-driven HDL development:

1. **Write C test programs** that exercise specific instructions, addressing modes, and edge cases. Compile with `v810-gcc` to raw binaries.
2. **Run each program in MAME's V810 emulator**, capturing an execution trace (register state per cycle or per instruction retire).
3. **Run the same program in our HDL simulation** (Verilator or iverilog), capturing the same trace.
4. **Diff traces.** Any mismatch is a bug. Start with simple programs, escalate to full CPU diagnostic suites.
5. **Bonus**: run Virtual Boy and PC-FX homebrew test programs ("vbtest", "pcfxcheck", etc.) once basic instructions work — these validate peripheral integration patterns even before we have peripherals.

This gives `v810-hdl` a sharp definition of done: "passes all tests bit-exact against MAME reference."

## Known V810 gotchas

Collected from community notes; verify each before relying on it:

- **Hardware interlocks, not software NOPs**, for pipeline hazards. The core stalls automatically on RAW dependencies.
- **FPU** is a real on-die component with its own exception conditions. Do not omit.
- **Bit-string instructions** (SCH0BSU, SCH1BSD, etc.) are multi-cycle and interruptible; state must be saved to registers mid-instruction so interrupts work cleanly.
- **Caching behavior** on uPD70732 (V810 variant used in PC-FX) differs from Virtual Boy's V810 (no cache). Matters for cycle-accuracy but can probably be ignored initially.
- **Endianness**: V810 is little-endian. Mind byte ordering in memory transactions.
- **Interrupt priorities**: nested-interrupt handling has quirks documented in the NEC manual; easy to get wrong.

## Open research items

- [ ] Acquire NEC user manual PDF, read cover to cover, extract instruction encoding tables into a machine-readable form (CSV or JSON) for the decoder.
- [ ] Build `v810-gcc` toolchain locally; verify it produces working binaries.
- [ ] Extract MAME's V810 test suite (if any) and any Virtual Boy / PC-FX homebrew test ROMs.
- [ ] Survey Planet Virtual Boy for any existing instruction test harnesses we can steal.
- [ ] Confirm which MAME version has the most current V810 code (occasional fixes land).
