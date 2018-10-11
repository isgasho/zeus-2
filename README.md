# zeus
Fictional cheap, Russian, electronic handheld game. 

## Technical specification

* 8-bit CPU
* 16-bit memory addresses
* 8kb of RAM (some mapped for specific purposes)
* 6 buttons (D-Pad, A, B)
* 2 audio channels (sin and noise, only one can be used at a time)
* 16x20 black and white screen (1 bit per pixel)
* Two 5 digit seven-segment displays for scores, etc
* Two 2 digit seven-segment displays for speed and level
* Games on ROM carts. Each cart can have up to 256 memory banks, and each bank can be up to 56kb long. 

## CPU

### Registers

| Register | Size | Purpose                                                                                                                          |
|----------|------|----------------------------------------------------------------------------------------------------------------------------------|
| X        | 8    | General purpose register.                                                                                                        |
| Y        | 8    | General purpose register.                                                                                                        |
| T        | 8    | General purpose register. Also stores the results of some operations.                                                            |
| PC       | 16   | Program counter. Initially set to 0x2000. This is where the attached ROM with a game begins (addresses 0x0000 - 0x1FFF are RAM). |


### Instruction set

All instruction names are 4 characters long.
All opcodes are 1-byte long, which means there is a maximum of 256 instructions, but most of these are unused. There may or may not be some undocumented instructions.

| Opcode | Instruction | Arguments                | Explanation                                                                                                                                                                                                                                                                                                                                                                           |
|--------|-------------|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0x00   | NOOP        | None                     | No op. Does nothing, proceeds to the next instruction as usual.                                                                                                                                                                                                                                                                                                                       |
| 0x01   | MVIX        | Immediate, 1 byte        | Move the next byte to register X.                                                                                                                                                                                                                                                                                                                                                     |
| 0x02   | MVIY        | Immediate, 1 byte        | Move the next byte to register Y.                                                                                                                                                                                                                                                                                                                                                     |
| 0x03   | MVIT        | Immediate, 1 byte        | Move the next byte to register T.                                                                                                                                                                                                                                                                                                                                                     |
| 0x04   | MVAX        | Addressed, 2 bytes       | Interpret the next two bytes as an address. Move the value at that address to X.                                                                                                                                                                                                                                                                                                      |
| 0x05   | MVAY        | Addressed, 2 bytes       | Same as above, but for Y.                                                                                                                                                                                                                                                                                                                                                             |
| 0x06   | MVAT        | Addressed, 2 bytes       | Same as above, but for T.                                                                                                                                                                                                                                                                                                                                                             |
| 0x07   | MVXA        | Addressed, 2 bytes       | Interpret the next two bytes as an address. Move the value of X to that address. If it's in ROM, ignore this instruction.                                                                                                                                                                                                                                                             |
| 0x08   | MVYA        | Addressed, 2 bytes       | Same as above, but for Y.                                                                                                                                                                                                                                                                                                                                                             |
| 0x09   | MVTA        | Addressed, 2 bytes       | Same as above, but for T.                                                                                                                                                                                                                                                                                                                                                             |
| 0x0A   | MVPA        | Addressed, 2 bytes       | Same as above, but for PC.                                                                                                                                                                                                                                                                                                                                                            |
| 0x0B   | ADDI        | See explanation, 6 bytes | Interpret the next 6 bytes as three 2-byte addresses. Take the byte at the first address, add it to the byte at the second address, and store it at the third address. If the third address is in ROM, ignore this instruction.                                                                                                                                                       |
| 0x0C   | SUBI        | See explanation, 6 bytes | Same as above, but subtract the second byte from the first and store it at the third address.                                                                                                                                                                                                                                                                                         |
| 0x0D   | MULI        | See explanation, 6 bytes | Same as above, but multiply the first two bytes and store it at the third address.                                                                                                                                                                                                                                                                                                    |
| 0x0E   | DIVI        | See explanation, 6 bytes | Same as above, but divide the first byte by the second and store it at the third address. Rounded down to nearest integer.                                                                                                                                                                                                                                                            |
| 0x0F   | MODI        | See explanation, 6 bytes | Same as above, but take the modulo of first/second and store it at the third address.                                                                                                                                                                                                                                                                                                 |
| 0x10   | SWIZ        | See explanation, 6 bytes | Interpret the next 6 bytes as three 2-byte addresses. Take the two bytes at the first address, apply the two bytes at the second address as a mask, and store the result at the third address. A value of 1-4 in the mask selects one of the digits of the first argument written as hex. Ex. Value: 0xABCD, mask: 0x4321, result: 0xDCBA. 0 and numbers 5-9 in the mask result in 0. |
| 0x11   | ANDI        | See explanation, 6 bytes | Same as ADDI, but apply logical AND to the first and second byte, then store it at the third address.                                                                                                                                                                                                                                                                                 |
| 0x12   | ORLI        | See explanation, 6 bytes | Same as above, but apply logical OR.                                                                                                                                                                                                                                                                                                                                                  |
| 0x13   | XORI        | See explanation, 6 bytes | Same as above, but apply logical XOR.                                                                                                                                                                                                                                                                                                                                                 |
| 0x14   | NEGI        | See explanation, 4 bytes | Interpret the next 4 bytes as two 2-byte addresses. Take the byte at the first address, negate it, and store it at the second address.                                                                                                                                                                                                                                                |
| 0x15   | SHLI        | See explanation, 4 bytes | As above, but shift the value left.                                                                                                                                                                                                                                                                                                                                                   |
| 0x16   | SHRI        | See explanation, 4 bytes | As above, but shift the value right.                                                                                                                                                                                                                                                                                                                                                  |
| 0x17   | EQLS        | Addressed, 4 bytes       | Compare the byte at the address given by the next two bytes to the one at the address given by the two bytes after that. If they're equal, set register T to 1.                                                                                                                                                                                                                       |
| 0x18   | GRTR        | Addressed, 4 bytes       | As above, but set T to 1 if the first byte is greater than the second.                                                                                                                                                                                                                                                                                                                |
| 0x19   | LESS        | Addressed, 4 bytes       | As above, but set T to 1 if the first byte is less than the second.                                                                                                                                                                                                                                                                                                                   |
| 0x20   | JUMP        | Immediate, 2 bytes       | Move the next two bytes to PC, moving execution to that address. Since there's no distinction between data and instructions, it can be anywhere in memory.                                                                                                                                                                                                                            |
| 0x21   | TJMP        | Immediate, 2 bytes       | As above, but jump only if T is not zero.                                                                                                                                                                                                                                                                                                                                             |
| 0x22   | FJMP        | Immediate, 2 bytes       | As above, but jump only if T is zero.                                                                                                                                                                                                                                                                                                                                                 |
| 0x23   | BANK        | Immediate, 1 byte        | Load the memory bank of current rom selected by the next byte, then set PC to 0x2000.                                                                                                                                                                                                                                                                                                 |
| 0x24   | RAND        | See explanation, 4 bytes | Interpret the next two bytes as lower and higher bound, then generate a random number between them (inclusive). Store it at the address given by the last two bytes.                                                                                                                                                                                                                  |
