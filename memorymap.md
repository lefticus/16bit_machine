# Memory Map

## Address Space Overview

Total address space: 64K words (0x0000 – 0xFFFF)

```
0x0000 – 0x000F    Zero Page          (16 words)
0x0010 – 0x00FF    System Reserved    (240 words)
0x0100 – 0x01FF    Hardware I/O       (256 words)
0x0200 – 0x7FFF    Program Space      (~32K words)
0x8000 – 0xFFFF    Banked Window      (32K words)
```

---

## Zero Page — 0x0000 to 0x000F

16 words of general RAM. Directly addressable by micro-immediate instructions via the zero-page indirect mode. Intended for frequently accessed variables, loop counters, and small constants that benefit from single-instruction access.

---

## System Reserved — 0x0010 to 0x00FF

240 words. Reserved for future system use. Reads return 0, writes are silently ignored.

---

## Hardware I/O — 0x0100 to 0x01FF

256 words of memory-mapped hardware registers. All device control is performed by reading and writing this region. Divided into 16 device slots of 16 words each.

```
0x0100 – 0x010F    Device  0  (system / bank control)
0x0110 – 0x011F    Device  1  (display)
0x0120 – 0x012F    Device  2  (timer)
0x0130 – 0x013F    Device  3  (interrupt controller)
0x0140 – 0x014F    Device  4  (reserved)
0x0150 – 0x015F    Device  5  (reserved)
0x0160 – 0x016F    Device  6  (reserved)
0x0170 – 0x017F    Device  7  (reserved)
0x0180 – 0x018F    Device  8  (reserved)
0x0190 – 0x019F    Device  9  (reserved)
0x01A0 – 0x01AF    Device 10  (reserved)
0x01B0 – 0x01BF    Device 11  (reserved)
0x01C0 – 0x01CF    Device 12  (reserved)
0x01D0 – 0x01DF    Device 13  (reserved)
0x01E0 – 0x01EF    Device 14  (reserved)
0x01F0 – 0x01FF    Device 15  (reserved)
```

Each device slot has 16 words available for control, status, and data registers. Device definitions are specified in separate device documentation. Reserved slots read as 0 and ignore writes.

---

## Program Space — 0x0200 to 0x7FFF

Approximately 32K words of fixed RAM. Code, data, and stack all live here.

### Suggested Layout

```
0x0200 – 0x02FF    System vectors and startup data
0x0300 – 0x6FFF    Program code and data
0x7000 – 0x7FFF    Stack (grows downward from 0x7FFF)
```

> Stack pointer (`sp`) should be initialized to 0x7FFF at startup. Stack grows downward, consistent with the descending stack convention.

---

## Banked Window — 0x8000 to 0xFFFF

32K words. Content is determined by the bank select register in Device 0. Allows access to additional RAM, video framebuffer, tile data, or other large memory-mapped resources without consuming fixed address space.

Bank definitions are specified in separate device and display documentation.
