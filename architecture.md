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
| Number of registers | 16 (2 hardwired, 2 system, 1 flags, 11 general purpose) |
| Instruction validity | All 2^16 encodings are valid and defined — no undefined behavior |
| Immediate load width | 8 bits per instruction |
| Primary operations | Register ↔ Register |
| Stack direction | Descending — `sp` decrements before write on push, increments after read on pop |

---

## Registers

| Encoding | Name | Purpose |
|---|---|---|
| `0000` | `r0` | Always 0 (hardwired) |
| `0001` | `r1` | Always 1 (hardwired) |
| `0010` | `ip` | Instruction pointer |
| `0011` | `sp` | Stack pointer |
| `0100` | `fl` | Flags (directly accessible) |
| `0101`–`1111` | `r5`–`r15` | General purpose (11 registers) |

---

## Flags Register

The flags register (`fl`) is directly readable and writable as a general purpose register. All 16 bits are defined — no undefined flag behavior.

| Bit | Flag | Name | Set When |
|---|---|---|---|
| 0 | `Z` | Zero | Result is zero |
| 1 | `N` | Negative | Result is negative (bit 15 set) |
| 2 | `C` | Carry | Unsigned overflow, or borrow on subtract |
| 3 | `V` | Overflow | Signed overflow, or result clamped |
| 4 | `FZ` | Float Zero | Float result underflowed to zero |
| 5 | `FN` | Float NaN | Float result is Not a Number |
| 6 | `FI` | Float Infinity | Float result is ±infinity |
| 7 | `FD` | Float Denormal | Float operation encountered a denormal input |
| 8 | `P` | Parity | XOR of all result bits is 1 (odd parity) |
| 9 | `S` | Saturated | `sadd` or `ssub` clamped the result |
| 10 | `IE` | Interrupt Enable | Interrupts are currently enabled |
| 11–15 | — | Reserved | Always 0, reserved for future use |

### Notes

- `fl` being a regular register means the entire flag state can be saved and restored with a single `mov` — useful for interrupt handlers and nested conditionals
- Float flags (`FZ`, `FN`, `FI`, `FD`) are sticky — they remain set until explicitly cleared, matching IEEE 754 exception flag behavior. This allows detection of exceptional conditions across multiple float operations
- `S` is distinct from `V` — saturation is intentional clamping, overflow is an error condition
- `IE` being in the flags register means enabling/disabling interrupts is a bitwise `or`/`and` operation on `fl`
- Reserved bits always read as 0 and writes are silently ignored

---



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

## Condition Codes

All unary instructions carry a 4-bit condition field (`cccc`). Conditions match the ARM condition code set exactly, in the same encoding order.

| Encoding | Mnemonic | Condition |
|---|---|---|
| `0000` | `eq` | Z set |
| `0001` | `ne` | Z clear |
| `0010` | `cs` | C set — unsigned higher or same |
| `0011` | `cc` | C clear — unsigned lower |
| `0100` | `mi` | N set — negative |
| `0101` | `pl` | N clear — positive or zero |
| `0110` | `vs` | V set — overflow |
| `0111` | `vc` | V clear — no overflow |
| `1000` | `hi` | C set and Z clear — unsigned higher |
| `1001` | `ls` | C clear or Z set — unsigned lower or same |
| `1010` | `ge` | N == V — signed greater or equal |
| `1011` | `lt` | N != V — signed less than |
| `1100` | `gt` | Z clear and N == V — signed greater than |
| `1101` | `le` | Z set or N != V — signed less or equal |
| `1110` | `al` | Always — unconditional |
| `1111` | `nv` | Never — executes as nop, useful for runtime patching |

> Signed comparisons (`ge`, `lt`, `gt`, `le`) depend on both N and V together. The flags register must track overflow (V) independently from sign (N).

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
| `11010` | — | Reserved |
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

## Opcode Reference

### Flag Key

| Symbol | Meaning |
|---|---|
| `✓` | Set based on result |
| `0` | Always cleared |
| `—` | Unaffected |
| `*` | See note |

Flags: **Z** = zero, **N** = negative, **C** = carry/borrow, **V** = signed overflow

---

### Defined Edge Cases

All operations have defined behavior in every case, including:

| Situation | Defined Behavior |
|---|---|
| `div` or `mod` by zero | V set, result is 0 |
| `fsqrt` of negative | V set, result is 0.0 |
| `ftoi` out of range | V set, result clamps to 0x7FFF (positive overflow) or 0x8000 (negative overflow) |
| `neg` of 0x8000 | V set, result is 0x8000 (no positive representation in 16-bit two's complement) |
| `abs` of 0x8000 | V set, result is 0x8000 (no positive representation in 16-bit two's complement) |
| `clz` of 0xFFFF | result is 0, Z set |
| `ctz` of 0xFFFF | result is 0, Z set |
| `shl`/`shr`/`sar` by 0 | result unchanged, C clear |
| `shl`/`shr`/`sar` by ≥16 | result is 0 (0xFFFF for `sar` on negative value), C clear |
| write to `r0` or `r1` | silently ignored, register retains hardwired value |

---



For float ops, C and V are not meaningful in the traditional integer sense. `fcmp` is the exception — it must set flags usefully for conditional branches to work on float results.

| Op | Result | Z | N | C | V | Notes |
|---|---|---|---|---|---|---|
| `fadd` | `dst ← dst + src` | ✓ | ✓ | — | — | |
| `fsub` | `dst ← dst - src` | ✓ | ✓ | — | — | |
| `fmul` | `dst ← dst × src` | ✓ | ✓ | — | — | |
| `fdiv` | `dst ← dst / src` | ✓ | ✓ | — | — | |
| `fmod` | `dst ← dst mod src` | ✓ | ✓ | — | — | |
| `fcmp` | flags from `dst - src` | ✓ | ✓ | * | — | C set if unordered (NaN involved) |
| `fmin` | `dst ← min(dst, src)` | ✓ | ✓ | — | — | |
| `fmax` | `dst ← max(dst, src)` | ✓ | ✓ | — | — | |

---

### Micro-Immediate Integer Operations

`#n` denotes the 4-bit immediate (0–15) or zero-page value.

| Op | Result | Z | N | C | V | Notes |
|---|---|---|---|---|---|---|
| `shl #n` | `dst ← dst << n` | ✓ | ✓ | * | — | C = last bit shifted out |
| `shr #n` | `dst ← dst >> n` | ✓ | ✓ | * | — | C = last bit shifted out, zero fill |
| `sar #n` | `dst ← dst >>> n` | ✓ | ✓ | * | — | C = last bit shifted out, sign fill |
| `add #n` | `dst ← dst + n` | ✓ | ✓ | ✓ | ✓ | |
| `sub #n` | `dst ← dst - n` | ✓ | ✓ | ✓ | ✓ | C = borrow |
| `and #n` | `dst ← dst & n` | ✓ | ✓ | 0 | 0 | |
| `cmp #n` | flags from `dst - n` | ✓ | ✓ | ✓ | ✓ | No result stored |
| `or #n`  | `dst ← dst \| n` | ✓ | ✓ | 0 | 0 | |

---

### Standard Integer ALU Operations

| Op | Result | Z | N | C | V | Notes |
|---|---|---|---|---|---|---|
| `add` | `dst ← dst + src` | ✓ | ✓ | ✓ | ✓ | |
| `addc` | `dst ← dst + src + C` | ✓ | ✓ | ✓ | ✓ | Includes carry in |
| `sub` | `dst ← dst - src` | ✓ | ✓ | ✓ | ✓ | C = borrow |
| `subb` | `dst ← dst - src - C` | ✓ | ✓ | ✓ | ✓ | C = borrow, includes borrow in |
| `mul` | `dst ← dst × src` | ✓ | ✓ | * | * | C and V set if result exceeds 16 bits |
| `div` | `dst ← dst / src` | ✓ | ✓ | 0 | * | V set on divide by zero |
| `mod` | `dst ← dst mod src` | ✓ | ✓ | 0 | * | V set on divide by zero |
| `and` | `dst ← dst & src` | ✓ | ✓ | 0 | 0 | |
| `or`  | `dst ← dst \| src` | ✓ | ✓ | 0 | 0 | |
| `xor` | `dst ← dst ^ src` | ✓ | ✓ | 0 | 0 | |
| `shl` | `dst ← dst << src` | ✓ | ✓ | * | — | C = last bit shifted out |
| `shr` | `dst ← dst >> src` | ✓ | ✓ | * | — | C = last bit shifted out, zero fill |
| `sar` | `dst ← dst >>> src` | ✓ | ✓ | * | — | C = last bit shifted out, sign fill |
| `cmp` | flags from `dst - src` | ✓ | ✓ | ✓ | ✓ | No result stored |
| `test` | flags from `dst & src` | ✓ | ✓ | 0 | 0 | No result stored |
| `mov` | `dst ← src` | ✓ | ✓ | — | — | |

---

### FPU Unary Operations

| Op | Result | Z | N | C | V | Notes |
|---|---|---|---|---|---|---|
| `fneg` | `dst ← -dst` | ✓ | ✓ | — | — | |
| `fabs` | `dst ← \|dst\|` | ✓ | 0 | — | — | Result always positive |
| `fsqrt` | `dst ← √dst` | ✓ | 0 | — | * | V set if input negative |
| `ftoi` | `dst ← (int)dst` | ✓ | ✓ | — | * | V set on out-of-range conversion |
| `itof` | `dst ← (float)dst` | ✓ | ✓ | — | — | |
| `ffloor` | `dst ← floor(dst)` | ✓ | ✓ | — | — | Float result |
| `fceil` | `dst ← ceil(dst)` | ✓ | ✓ | — | — | Float result |
| `fround` | `dst ← round(dst)` | ✓ | ✓ | — | — | Float result |

---

### ALU Unary Operations

| Op | Result | Z | N | C | V | Notes |
|---|---|---|---|---|---|---|
| `jmp` | `ip ← dst` | — | — | — | — | |
| `jmprelative` | `ip ← ip + dst` | — | — | — | — | |
| `call` | `[sp] ← ip, sp ← sp-1, ip ← dst` | — | — | — | — | |
| `callrelative` | `[sp] ← ip, sp ← sp-1, ip ← ip+dst` | — | — | — | — | |
| `ret` | `sp ← sp+1, ip ← [sp]` | — | — | — | — | |
| `push` | `sp ← sp-1, [sp] ← dst` | — | — | — | — | |
| `pop` | `dst ← [sp], sp ← sp+1` | — | — | — | — | |
| `not` | `dst ← ~dst` | ✓ | ✓ | 0 | 0 | |
| `rol` | `dst ← rol(dst)` | ✓ | ✓ | * | — | C = bit 15 rotated out |
| `ror` | `dst ← ror(dst)` | ✓ | ✓ | * | — | C = bit 0 rotated out |
| `clz` | `dst ← count_leading_zeros(dst)` | ✓ | 0 | — | — | Z set if dst was 0xFFFF |
| `ctz` | `dst ← count_trailing_zeros(dst)` | ✓ | 0 | — | — | Z set if dst was 0xFFFF |
| `popcount` | `dst ← count_set_bits(dst)` | ✓ | 0 | — | — | Result always 0–16 |
| `bswap` | `dst ← swap_bytes(dst)` | ✓ | ✓ | — | — | Swaps high and low bytes |
| `neg` | `dst ← -dst` | ✓ | ✓ | ✓ | * | V set if dst was 0x8000 |
| `abs` | `dst ← \|dst\|` | ✓ | 0 | — | * | V set if dst was 0x8000 |
| `sext` | `dst ← sign_extend(dst[7:0])` | ✓ | ✓ | — | — | Extends low byte to 16 bits |
| `set` | `dst ← 1` | — | — | — | — | Predicate controls execution, not result |
| `halt` | stop execution | — | — | — | — | |

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
