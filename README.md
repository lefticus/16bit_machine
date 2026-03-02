# 16 bit machine

This is a machine architecture design and test suite for a pure 16 bit CPU and associated computer.

# Architecture

Pure 16 bit design.

* Registers are 16 bits
* Ram is 16 bits
* Address space is 16 bits
* All instructures are exactly 1 word (16 bits)
* All 2^16 instructions are valid and defined behavior
* Immediate loads are 8 bits at a time, by necessity
* There are 16 registers
* Most operations are register<->register operations



* opcode: 4bits
* register (src/dst): 4 bits
* payload: 8 bits
* extended: 4 bits
   * 0: dst indirect
   * 1: src indirect
   * 2-3: opcode sub options
 
of top 8 bits:


1100 0000 - 1111 1111 - are valid load opcodes       (192 - 255)
0000 0000 - 1011 1111 - are the rest of the opcodes  (0 - 191)

```
//  im load   11xx
//
// dst indirect V
//  low/high     V
// load       11xx dst payload

```

0 - 191 - remaining opcodes

```
# ALU/FPU modes opcode dst src
#            xx00 00yz dst src
#
# xx can be 00, 01, 10
#
# y dst indirect
# z src indirect
#
# 48 opcodes remaining
```

binary

bitwise

xor
or
and

arithmetic 

add
addc
subc
fdiv
fadd
sub
div
jmp
cmp 
shl
shr
sar
rol
ror


unary

not
jmp <16 types of jumps because of available register>
  relative
  absolute
  

## Decoding

 * 11xx xxxx xxxx xxxx - load operation
    * 110x xxxx xxxx xxxx - direct into register
    * 111x xxxx xxxx xxxx - indirect to address pointed to by register
        * 11i0 xxxx xxxx xxxx - load of low byte
        * 11i1 xxxx xxxx xxxx - load of high byte
            * 11ib rrrr pppp pppp - load payload
         
