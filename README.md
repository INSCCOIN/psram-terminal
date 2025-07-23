# PicoCalc PSRAM Monitor

## Overview

This project implements a **Soft SPI-based PSRAM monitor for Raspberry Pi Pico running PicoMite MMBasic (v6.0002 - v6.0003)**.
It allows direct **read, write, dump, fill, and checksum operations** on external PSRAM (e.g., ESP-PSRAM32) using a simple terminal interface.

It is designed for **limited GPIO availability**, working only with these allowed pins:
- Pin 4 → GP2 (CS)
- Pin 5 → GP3 (SCK)
- Pin 6 → GP4 (MOSI)
- Pin 7 → GP5 (MISO)
- SIO2 and SIO3 of PSRAM must be **pulled HIGH** for single-bit SPI mode.

---

## Hardware Context

- MCU: Raspberry Pi Pico (PicoMite MMBasic v6.0002)
- External RAM: ESP-PSRAM32 or compatible SPI PSRAM
- Interface: Soft SPI at 1 MHz, Mode 0 (CPOL=0, CPHA=0)
- Addressing: 24-bit addresses, supports up to 1MB memory range
- Protocol:
  - 0x02 → WRITE
  - 0x03 → READ

---

## Features Implemented

- Single-byte read/write to any PSRAM address
- Block fill with a constant value
- Hex dump for visualizing memory contents
- Pointer-based stepping (set pointer + dump relative)
- 8-bit checksum calculation over arbitrary memory regions

Planned extensions:
- CRC16/CRC32 for stronger integrity checks
- Memory compare tool (NOT COMMAND)

---

## Commands

| Command                 | Example               | Description |
|-------------------------|-----------------------|-------------|
| `W <addr> <val>`        | `W 0 123`             | Write 1 byte to address 
| `R <addr>`              | `R 0`                 | Read 1 byte from address 
| `F <start> <end> <val>` | `F 0 255 170`         | Fill memory block with value 
| `D <start> <len>`       | `D 0 64`              | Dump memory as hex 
| `S <addr>`              | `S 512`               | Set current memory pointer 
| `N <len>`               | `N 32`                | Dump from pointer & advance 
| `C <start> <len>`       | `C 0 256`             | Compute checksum of a memory block 

---

## Example Usage

```
> F 0 255 170
Filled 000000 to 0000FF with 170

> D 0 64
000000: AA AA AA AA AA AA AA AA AA AA AA AA AA AA AA AA 
000010: AA AA AA AA AA AA AA AA AA AA AA AA AA AA AA AA

> W 0 1
Wrote 1 to 000000

> R 0
Read 1 (01) from 000000

> C 0 256
Checksum [000000..0000FF] = 01
```

---

## Why Soft SPI?

The PicoMite’s hardware SPI is fixed to certain pins, but this project is constrained to only **pins 4, 5, 6, and 7**.
Hence, we use **software bit-banged SPI** so any GPIOs can be used for SCK/MOSI/MISO.

---

## How It Works Internally

- CS LOW → send command (0x02/0x03) → send 3-byte address → read/write data → CS HIGH
- For WRITE:
  - Send 0x02 + [AddrHi AddrMid AddrLow] + DataByte
- For READ:
  - Send 0x03 + [AddrHi AddrMid AddrLow] then read 1 byte

Each monitor command wraps these SPI transactions:
- `W` calls PSRAM_WriteByte()
- `R` calls PSRAM_ReadByte()
- `F` loops multiple writes
- `D` loops multiple reads and formats them in hex
- `C` loops multiple reads and accumulates checksum

---

## Future Ideas

- Burst-write for faster filling  
- Block-compare utility  
- CRC32 for robust verification  
- Auto self-test pattern generator  

---

## License

INSCCOIN Public License v1.0
