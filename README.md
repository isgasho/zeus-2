# zeus
Fictional cheap, Russian, electronic handheld game. 

![screenshot](https://i.imgur.com/ciLLM9T.png)

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

The cpu is intended to perform at a fixed speed of processing 65535 operations between every two frames. This makes the device state portable, and, excluding the button inputs, completely deterministic.


### Registers

| Register | Size | Purpose                                                                                                                          |
|----------|------|----------------------------------------------------------------------------------------------------------------------------------|
| X        | 8    | General purpose register.                                                                                                        |
| Y        | 8    | General purpose register. Also used for indirect addressing.                                                                     |
| T        | 8    | General purpose register. Also stores the results of some operations.                                                            |
| S        | 8    | Status register. Interpreted as 8 1-bit flags. Reserved for future use.                                                          |
| PC       | 16   | Program counter. Initially set to 0x2000. This is where the attached ROM with a game begins (addresses 0x0000 - 0x1FFF are RAM). |
| N        | 16   | Instruction count for current frame.                                                                                             |


### Operation

The following steps define the operation of the CPU:
* If N == 0xFFFF (last instruction for this frame):
  * Sync screen with memory
* Otherwise:
  * Read the byte at the address stored in PC
  * Increment PC
  * Execute the instruction identified by this opcode
* Increment N
* Go to start

### Instruction set

All instruction names are 4 characters long.
All opcodes are 1-byte long, which means there is a maximum of 256 instructions, but most of these are unused. There may or may not be some undocumented instructions.

| Opcode | Instruction | Arguments                    | Explanation                                                                                                                                                                                                                                                                                                                                                                           |
|--------|-------------|------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0x00   | NOOP        | None                         | No op. Does nothing, proceeds to the next instruction as usual.                                                                                                                                                                                                                                                                                                                       |
| 0x01   | MVIX        | Immediate, 1 byte            | Move the next byte to register X.                                                                                                                                                                                                                                                                                                                                                     |
| 0x02   | MVIY        | Immediate, 1 byte            | Move the next byte to register Y.                                                                                                                                                                                                                                                                                                                                                     |
| 0x03   | MVIT        | Immediate, 1 byte            | Move the next byte to register T.                                                                                                                                                                                                                                                                                                                                                     |
| 0x04   | MVAX        | Addressed, 2 bytes           | Interpret the next two bytes as an address. Move the value at that address to X.                                                                                                                                                                                                                                                                                                      |
| 0x05   | MVAY        | Addressed, 2 bytes           | Same as above, but for Y.                                                                                                                                                                                                                                                                                                                                                             |
| 0x06   | MVAT        | Addressed, 2 bytes           | Same as above, but for T.                                                                                                                                                                                                                                                                                                                                                             |
| 0x07   | MVXA        | Addressed, 2 bytes           | Interpret the next two bytes as an address. Move the value of X to that address. If it's in ROM, ignore this instruction.                                                                                                                                                                                                                                                             |
| 0x08   | MVYA        | Addressed, 2 bytes           | Same as above, but for Y.                                                                                                                                                                                                                                                                                                                                                             |
| 0x09   | MVTA        | Addressed, 2 bytes           | Same as above, but for T.                                                                                                                                                                                                                                                                                                                                                             |
| 0x0A   | MVPA        | Addressed, 2 bytes           | Same as above, but for PC.                                                                                                                                                                                                                                                                                                                                                            |
| 0x0B   | ADDX        | Immediate, 1 byte            | Add the next byte to X.                                                                                                                                                                                                                                                                                                                                                               |
| 0x0C   | ADDY        | Immediate, 1 byte            | Add the next byte to Y.                                                                                                                                                                                                                                                                                                                                                               |
| 0x0D   | ADDT        | Immediate, 1 byte            | Add the next byte to T.                                                                                                                                                                                                                                                                                                                                                               |
| 0x0E   | SUBX        | Immediate, 1 byte            | Subtract the next byte from X.                                                                                                                                                                                                                                                                                                                                                        |
| 0x0F   | SUBY        | Immediate, 1 byte            | Subtract the next byte from Y.                                                                                                                                                                                                                                                                                                                                                        |
| 0x10   | SUBT        | Immediate, 1 byte            | Subtract the next byte from T.                                                                                                                                                                                                                                                                                                                                                        |
| 0x11   | COPY        | Immediate/Addressed, 3 bytes | Take the next byte and copy it to the address given by the two bytes after it.                                                                                                                                                                                                                                                                                                        |
| 0x12   | CPID        | Indirect addressed, 4 bytes  | Indirect copy. Interpret the next two bytes as an address, and copy the value at that address to the address given by adding the address two bytes after it to the value in Y register.                                                                                                                                                                                               |
| 0x13   | CPIR        | Indirect addressed, 4 bytes  | Indirect copy, reversed. Interpret the next two bytes as an address, and add the value of the Y register to it. Copy the value at that address to the address given by the next two bytes.                                                                                                                                                                                            |
| 0x14   | ADDI        | See explanation, 6 bytes     | Interpret the next 6 bytes as three 2-byte addresses. Take the byte at the first address, add it to the byte at the second address, and store it at the third address. If the third address is in ROM, ignore this instruction.                                                                                                                                                       |
| 0x15   | SUBI        | See explanation, 6 bytes     | Same as above, but subtract the second byte from the first and store it at the third address.                                                                                                                                                                                                                                                                                         |
| 0x16   | MULI        | See explanation, 6 bytes     | Same as above, but multiply the first two bytes and store it at the third address.                                                                                                                                                                                                                                                                                                    |
| 0x17   | DIVI        | See explanation, 6 bytes     | Same as above, but divide the first byte by the second and store it at the third address. Rounded down to nearest integer.                                                                                                                                                                                                                                                            |
| 0x18   | MODI        | See explanation, 6 bytes     | Same as above, but take the modulo of first/second and store it at the third address.                                                                                                                                                                                                                                                                                                 |
| 0x19   | SWIZ        | See explanation, 6 bytes     | Interpret the next 6 bytes as three 2-byte addresses. Take the two bytes at the first address, apply the two bytes at the second address as a mask, and store the result at the third address. A value of 1-4 in the mask selects one of the digits of the first argument written as hex. Ex. Value: 0xABCD, mask: 0x4321, result: 0xDCBA. 0 and numbers 5-9 in the mask result in 0. |
| 0x1A   | ANDI        | See explanation, 6 bytes     | Same as ADDI, but apply logical AND to the first and second byte, then store it at the third address.                                                                                                                                                                                                                                                                                 |
| 0x1B   | ORLI        | See explanation, 6 bytes     | Same as above, but apply logical OR.                                                                                                                                                                                                                                                                                                                                                  |
| 0x1C   | XORI        | See explanation, 6 bytes     | Same as above, but apply logical XOR.                                                                                                                                                                                                                                                                                                                                                 |
| 0x1D   | NEGI        | See explanation, 4 bytes     | Interpret the next 4 bytes as two 2-byte addresses. Take the byte at the first address, negate it, and store it at the second address.                                                                                                                                                                                                                                                |
| 0x1E   | SHLI        | See explanation, 4 bytes     | As above, but shift the value left.                                                                                                                                                                                                                                                                                                                                                   |
| 0x1F   | SHRI        | See explanation, 4 bytes     | As above, but shift the value right.                                                                                                                                                                                                                                                                                                                                                  |
| 0x20   | EQLS        | Addressed, 4 bytes           | Compare the byte at the address given by the next two bytes to the one at the address given by the two bytes after that. If they're equal, set register T to 1.                                                                                                                                                                                                                       |
| 0x21   | GRTR        | Addressed, 4 bytes           | As above, but set T to 1 if the first byte is greater than the second.                                                                                                                                                                                                                                                                                                                |
| 0x22   | LESS        | Addressed, 4 bytes           | As above, but set T to 1 if the first byte is less than the second.                                                                                                                                                                                                                                                                                                                   |
| 0x23   | JUMP        | Immediate, 2 bytes           | Move the next two bytes to PC, moving execution to that address. Since there's no distinction between data and instructions, it can be anywhere in memory.                                                                                                                                                                                                                            |
| 0x24   | TJMP        | Immediate, 2 bytes           | As above, but jump only if T is not zero.                                                                                                                                                                                                                                                                                                                                             |
| 0x25   | FJMP        | Immediate, 2 bytes           | As above, but jump only if T is zero.                                                                                                                                                                                                                                                                                                                                                 |
| 0x26   | RJMP        | Immediate, 3 bytes           | Relative jump. The first byte after this instruction will be interpreted as a sign (negaitve if 0, positive otherwise). Then, execution will jump backwards or forwards by the number of bytes read from the subsequent 2 bytes depending on that sign.                                                                                                                               |
| 0x27   | BANK        | Immediate, 1 byte            | Load the memory bank of current rom selected by the next byte, then set PC to 0x2000.                                                                                                                                                                                                                                                                                                 |
| 0x28   | RAND        | See explanation, 4 bytes     | Interpret the next two bytes as lower and higher bound, then generate a random number between them (inclusive). Store it at the address given by the last two bytes.                                                                                                                                                                                                                  |
| 0x29   | WAIT        | None                         | Pause execution for the remainder of the current frame. Sets N to 0xFFFE, which causes a sync in the next cycle, and waits for the beginning of the next frame.                                                                                                                                                                                                                       |
| 0x2A   | CLRS        | None                         | Clear the screen memory.                                                                                                                                                                                                                                                                                                                                                              |

## Memory

The device has a RAM module with 8kb of memory, mapped to addresses 0x0000 - 0x1FFF. Some of these addresses are reserved for controlling inputs and outputs.

When a ROM with a game is loaded, the memory bank number 0 is attached to the device's memory so that it can be addressed by the range 0x2000 - 0xFFFF. This means that a memory bank has a maximum size of 56kb. If it's smaller, it will be padded with zeroes. Any write operations to addresses higher than 0x1FFF will be ignored.

The table below lists reserved memory areas and their purpose.


| Range       | Purpose           | Details                                                                                                                                                                                                                                                                                                                                                                                                                            |
|-------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0x0000 - 0x0027 | Screen control    | The contents of these 40 bytes control the display. At the end of each frame, the screen is synchronized with this memory range by interpreting subsequent bits as pixels, row by row.                                                                                                                                                                                                                                             |
| 0x0028        | Button inputs     | The device's buttons are stored in a single byte at this address. Each bit represents a single button and it is flipped as the button is pressed and released. 1 means a button is down, 0 means up. The 0th bit is the left d-pad button, 1st - up, 2nd - right, 3rd - down, 4th - A, 5th - B. 6th and 7th are unused. Writing to this memory has no effect, and it will be overwritten the next time any button's state changes. |
| 0x0029 - 0x0035 | 7-segment display | There are two 5-digit displays, and two 2-digit displays, for a total of 14 digits. This memory area controls their state, with each byte representing one digit.                                                                                                                                                                                                                                                                  |

## ROMs

ROMs are binary files comprising of a header and game data.
The structure of the header is as follows:

| Field name   | Size (bytes) | Description                                                                                                                                |
|--------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| Magic number | 4            | Magic number so that the program can confirm it's the correct file format. Must always be equal to 0x5A 0x45 0x0x55 0x53 (uppercase zeus). |
| Version      | 3            | Version of the file normat. The subsequent bytes represent the major, minor, and patch version.                                            |
| Reserved     | 25           | Reserved for future use, padding the header to 32 bytes total. Ignored for now, should be filled with zeroes.                              |

After the header section comes the game data. It's read as is and padded with zeroes to 56kb. The game data will be split into 56kb memory banks (up to 256) if there's more data in the file. In this case, the last bank will be padded.

## Rendering

No dedicated graphics unit.
As described in the RAM section, memory range 0x0000 - 0x0027 is mapped onto the screen. The contents of memory in this section are synchronized with screen state 30 times per second, instantly, between cpu instructions (so there's no way to change anything between rendering phases).
