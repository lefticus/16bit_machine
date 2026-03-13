# 16-Bit Machine

An educational 16-bit CPU architecture and computer design. The goal is a clean, well-defined, and approachable architecture suitable for teaching computer organization, assembly programming, and systems concepts.

---

## Design Principles

- Every instruction is exactly one 16-bit word
- All 2^16 instruction encodings are valid — no undefined behavior
- All operations are register-to-register with universal indirect addressing
- No segmentation — flat 32K program space with a banked upper window
- Conditions are ARM-compatible and available on all unary instructions
- Floating point is a first-class citizen alongside integer operations

---

## Documents

### [architecture.md](architecture.md)
The core CPU specification. Covers registers, instruction encoding, the full opcode table with result and flag behavior, condition codes, and the flags register layout.

### [memorymap.md](memorymap.md)
The system memory map. Covers zero page, hardware I/O device slots, program space layout, and the banked upper window.

---

## Status

Early design phase. Opcode set and memory map are largely settled. The following areas are still being defined:

- Display hardware and video modes
- Device register specifications
- Calling convention
- Assembler syntax
