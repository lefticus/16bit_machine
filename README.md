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

Register allocation

tt - opcode type

0? binary opcodes

00 ii 0000
00 ii 0111 - FPU

fdiv
fmul
fmod
fadd
fcmp
fsub
fmin
fmax

00 ii 1??? - Immediate mode binary micro ops

00 ii 1000
00 ii 1111 - Immediate mode micro ops

shl
shr
sar
add
sub
and
cmp
or

01 ii ???? - 

01 ii 0000
01 ii 1111 - ALU ops

xor
addc
subb
mul
div
mod
test
mov

shl
shr
sar
add
sub
and
cmp
or


10 unary opcodes

10 io oooo dddd cccc - FPU

(0-7, conditional)

fneg
fabs
fsqrt
ftoi
itof
ffloor
fceil
fround

10 io oooo dddd cccc - ALU

(8-27, ALU, conditional)

jmp
jmprelative
call
callrelative
ret
push
pop
not
rol
ror
clz
ctz
popcount
bswap
neg
abs
sext
set
nop
halt

(28-31, GPU, reserved)




11 prefix - immediate load opcodes

11 ib rrrr pppppppp

11 i0 load of low byte
11 i1 load of high byte


registers:

0000 - always 0
0001 - always 1
0010 - ip
0011 - sp
0100 - instruction segment
0101 - address segment
0110 - stack segment
0111 - flags


## Decoding

 * 11xx xxxx xxxx xxxx - load operation
    * 110x xxxx xxxx xxxx - direct into register
    * 111x xxxx xxxx xxxx - indirect to address pointed to by register
        * 11i0 xxxx xxxx xxxx - load of low byte
        * 11i1 xxxx xxxx xxxx - load of high byte
            * 11ib rrrr pppp pppp - load payload
         
