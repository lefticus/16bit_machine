# 16-Bit Machine Architecture

A machine architecture design and test suite for a pure 16-bit CPU and associated computer.

---

## Overview

| Property | Value |
|---|---|
| Register width | 16 bits |
| RAM width | 16 bits |
| Address space | 16 bits (64K words) |
| Instruction width | 16 bits (exactly 1 word) |
| Number of registers | 16 |
| Instruction validity | All 2^16 encodings are valid and defined |
| Immediate load width | 8 bits per instruction |
| Primary operations | Register ↔ Register |

---

## Registers

| Encoding | Name | Purpose |
|---|---|---|
| `0000` | `r0` | Always 0 (hardwired) |
| `0001` | `r1` | Always 1 (hardwired) |
| `0010` | `ip` | Instruction pointer |
| `0011` | `sp` | Stack pointer |
| `0100` | `is` | Instruction segment |
| `0101` | `as` | Address segment |
| `0110` | `ss` | Stack segment |
| `0111` | `fl` | Flags (directly accessible) |
| `1000`–`1111` | `r8`–`r15` | General purpose |

---

## Instruction Encoding

The top 2 bits (`tt`) define the instruction type:

| `tt` | Type |
|---|---|
| `00` | Binary opcodes |
| `01` | Binary opcodes (continued) |
| `10` | Unary opcodes |
| `11` | Immediate load |

---

## Binary Opcodes (`00`, `01`)

Binary instructions operate on two registers. Every register access supports direct or indirect addressing.

### Binary Encoding

```
tt d i oooo dddd ssss
   │ │ ││││ ││││ ││││
   │ │ └┴┴┴ opcode (4 bits)
   │ └ src indirect flag
   └ dst indirect flag
           dddd = destination register (4 bits)
           ssss = source register or immediate (4 bits)
```

| Bit | Field | Meaning |
|---|---|---|
| `tt` | type | `00` = FPU / micro-immediate, `01` = standard ALU |
| `d` | dst indirect | 0 = register direct, 1 = address pointed to by register |
| `i` | src indirect | 0 = register or immediate direct, 1 = indirect |
| `oooo` | opcode | selects operation within group |
| `dddd` | destination | target register |
| `ssss` | source | source register or 4-bit immediate |

### Zero Page in Micro-Immediate Mode

In micro-immediate instructions the source field is a 4-bit value (0–15) rather than a register. The `i` (src indirect) bit then has a special meaning:

```
i = 0  →  ssss IS the immediate value (0–15)
i = 1  →  ssss is an ADDRESS into zero page (first 16 words of address space)
```

This gives micro-immediate ops access to 16 dedicated zero-page memory locations — similar to the 6502's zero page — without consuming a register. Frequently used constants or variables can be placed there and accessed directly in a single instruction.

---

### FPU Operations — `00 ii 0000` to `00 ii 0111`

| Encoding | Op | Description |
|---|---|---|
| `000` | `fdiv` | Float divide |
| `001` | `fmul` | Float multiply |
| `010` | `fmod` | Float modulo |
| `011` | `fadd` | Float add |
| `100` | `fcmp` | Float compare |
| `101` | `fsub` | Float subtract |
| `110` | `fmin` | Float minimum |
| `111` | `fmax` | Float maximum |

---

### Micro-Immediate Integer Operations — `00 ii 1000` to `00 ii 1111`

Source operand is a 4-bit immediate value (0–15) instead of a register.

| Encoding | Op | Description |
|---|---|---|
| `000` | `shl` | Shift left |
| `001` | `shr` | Shift right logical |
| `010` | `sar` | Shift right arithmetic |
| `011` | `add` | Add |
| `100` | `sub` | Subtract |
| `101` | `and` | Bitwise AND |
| `110` | `cmp` | Compare |
| `111` | `or` | Bitwise OR |

---

### Standard Integer ALU Operations — `01 ii 0000` to `01 ii 1111`

| Encoding | Op | Description |
|---|---|---|
| `0000` | `xor` | Bitwise XOR |
| `0001` | `addc` | Add with carry |
| `0010` | `subb` | Subtract with borrow |
| `0011` | `mul` | Multiply |
| `0100` | `div` | Divide |
| `0101` | `mod` | Modulo |
| `0110` | `test` | Bitwise AND, sets flags only |
| `0111` | `mov` | Move |
| `1000` | `shl` | Shift left |
| `1001` | `shr` | Shift right logical |
| `1010` | `sar` | Shift right arithmetic |
| `1011` | `add` | Add |
| `1100` | `sub` | Subtract |
| `1101` | `and` | Bitwise AND |
| `1110` | `cmp` | Compare |
| `1111` | `or` | Bitwise OR |

---

## Unary Opcodes (`10`)

Unary instructions operate on a single register and carry a 4-bit predicate field for conditional execution.

```
10 i ooooo dddd cccc
   │ │││││ ││││ ││││
   │ └┴┴┴┴ opcode (5 bits)
   └ indirect flag (1 bit)
           dddd = destination register (4 bits)
           cccc = predicate condition (4 bits)
```

> The extra bit in the predicate field enables an extended predicate mode: when set, the predicate is sourced from a general-purpose register rather than the flags register, allowing stored conditions to be evaluated without disturbing flags.

---

### FPU Unary Operations — opcodes `0`–`7`

| Opcode | Op | Description |
|---|---|---|
| `00000` | `fneg` | Float negate |
| `00001` | `fabs` | Float absolute value |
| `00010` | `fsqrt` | Float square root |
| `00011` | `ftoi` | Float to integer |
| `00100` | `itof` | Integer to float |
| `00101` | `ffloor` | Floor (float result) |
| `00110` | `fceil` | Ceiling (float result) |
| `00111` | `fround` | Round (float result) |

---

### ALU Unary Operations — opcodes `8`–`27`

| Opcode | Op | Description |
|---|---|---|
| `01000` | `jmp` | Absolute jump |
| `01001` | `jmprelative` | Relative jump |
| `01010` | `call` | Absolute call |
| `01011` | `callrelative` | Relative call |
| `01100` | `ret` | Return |
| `01101` | `push` | Push to stack |
| `01110` | `pop` | Pop from stack |
| `01111` | `not` | Bitwise NOT |
| `10000` | `rol` | Rotate left |
| `10001` | `ror` | Rotate right |
| `10010` | `clz` | Count leading zeros |
| `10011` | `ctz` | Count trailing zeros |
| `10100` | `popcount` | Count set bits |
| `10101` | `bswap` | Byte swap |
| `10110` | `neg` | Integer negate |
| `10111` | `abs` | Integer absolute value |
| `11000` | `sext` | Sign extend |
| `11001` | `set` | Set register to 1 (materializes predicate) |
| `11010` | `nop` | No operation |
| `11011` | `halt` | Halt execution |

---

### GPU Operations — opcodes `28`–`31` *(reserved)*

| Opcode | Op | Description |
|---|---|---|
| `11100` | — | Reserved |
| `11101` | — | Reserved |
| `11110` | — | Reserved |
| `11111` | — | Reserved |

---

## Immediate Load Opcodes (`11`)

Loads an 8-bit immediate into the low or high byte of a register, direct or indirect.

```
11 i b rrrr pppp pppp
   │ │ ││││ ││││ ││││
   │ │ └┴┴┴ destination register (4 bits)
   │ └ byte select: 0 = low byte, 1 = high byte
   └ indirect flag: 0 = direct, 1 = indirect (address in register)

pppp pppp = 8-bit immediate payload
```

| Prefix | Mode |
|---|---|
| `110` | Direct load into register |
| `111` | Indirect load to address pointed to by register |

### Decoding Tree

```
11xx xxxx xxxx xxxx  →  Immediate load
├── 110x xxxx xxxx xxxx  →  Direct
│   ├── 1100 rrrr pppp pppp  →  Load low byte
│   └── 1101 rrrr pppp pppp  →  Load high byte
└── 111x xxxx xxxx xxxx  →  Indirect
    ├── 1110 rrrr pppp pppp  →  Load low byte to [reg]
    └── 1111 rrrr pppp pppp  →  Load high byte to [reg]
```
