---
title: Exploring CHIP-8 Emulator [C++]
category: Software/Explore
---

A simple exploration into combining software and hardware.

Emulators have been a big part of my childhood. I used to download and play a lot of Pokemon games through the [Desmume](https://github.com/TASEmulators/desmume) emulator wondering how I could play this game without the actual machine. Exploring through the making of an emulator gave me a better understanding of both software and hardware overall, which acted as a connection between the two. Through this article, I am exploring through the core aspects of how CHIP-8 and emulators work overall, and how to recreate the process through software alone. This article is heavily influnced by Austin Morlan's [Building a CHIP-8 Emulator](https://austinmorlan.com/posts/chip8_emulator/), which gave me the initial push to write this.

# Emulators
Emulators are software replications of physical electronic devices, such as the Nintendo Game Boy. Early gaming hardware lacked the computing power to render advanced graphics, and developers couldn't offload substantial processing from the slow CPU. Consequently, iconic early games like Super Mario Bros. had to be heavily optimized just to fit onto these limited systems. Today, thanks to exponential advancements in technology, modern computers possess the processing power to run hundreds of Game Boy games simultaneously. This massive increase of power makes it easy to replicate the core functions of vintage hardware inside a modern software environment.

If you play a lot of video games, the most common hurdle you will run into is compatibility. You might own a PlayStation console but want to play an Xbox game, or vice versa. Unlike modern consoles—which are essentially specialized PCs under the hood—early gaming hardware was built from scratch using wildly different architectural philosophies. Console manufacturers constantly diversified their hardware formats to optimize speed, capacity, and cost, as well as to implement anti-piracy measures. This fragmentation eventually forced the market to consolidate around a few dominant platform holders, like Nintendo and Sony. Because games were coded directly for these unique hardware layouts, they were inherently incompatible with rival systems.

This compatibility barrier persisted throughout the growth of the gaming industry, forcing developers to dedicate their software to specific hardware environments. Although the industry has recently moved toward unifying these vast markets, we still cannot natively run the vast legacy of classic software on modern systems. To bridge this gap, developers have leveraged the enhanced processing power of modern general-purpose computers to build software copies of these vintage systems, creating what we know today as the emulator.

# Hardware
## CHIP-8
In the late 1970s, microcomputers had tiny amounts of RAM (often just 2KB to 4KB) and slow processors. Writing a game required writing raw assembly language or machine code specific to that computer's exact processor chip. This was incredibly tedious and difficult.  

Joseph Weisbecker wrote a tiny 512-byte interpreter program (a virtual machine) that sat in the computer's memory. This interpreter ran a much simpler, universal language he designed: CHIP-8. Instead of dealing with complex hardware code, hobbyists could write games using CHIP-8's straightforward commands, and the interpreter would translate them on the fly.

Here are the general specs of the CHIP-8:

1. 4,096 bytes (4KB) of addressable RAM, indexed from 0x000 to 0xFFF.

0x000 to 0x1FF: Originally reserved for the interpreter itself. In modern emulators, this space is typically empty and used to store font data.
0x200 to 0xFFF: Where the actual game data (ROM) is loaded and executed.

2. 16 general purpose 8-bit registers (V0 through VF).

A register is basically a location on a CPU for storage. All instructions and operations must be done within the resgisters. Since it is 8-bit, that means each register is only able to hold any value from 0x00 to 0xFF. The last register (VF) is a "flag" register used to handle math carries and graphics collisions, basically an error flag.

3. Index 16-bit register (I) to store memory addresses.

4. Program counter (16-bit pointer) that tracks the memory address of the current instruction.

5. Stack

A small, dedicated array (or block of memory) used exclusively to keep track of execution flow during subroutines (function calls). When a program executes a CALL instruction to run a different block of code, it saves the current Program Counter address into this stack. This ensures the CPU knows exactly where to resume once the subroutine is finished.

6. Stack pointer

An 8-bit variable that tracks the current active index of the stack array. When a new return address is added (pushed) to the stack, the stack pointer increments. When an address is removed (popped) during a RETURN instruction, the stack pointer decrements.

7. Delay and Sound timers

These are two independent countdown clocks. If you set them to any number above 0, they will automatically count down at a rate of 60 times per second (60Hz).

8. Input keys

The original CHIP-8 computers used a 16-button hexadecimal keypad, labeled 0 through F. When building an emulator on a modern computer, you usually map these 16 inputs to a cluster of keys on your QWERTY keyboard (for example, mapping them to the 1-4, Q-R, A-F, and Z-V keys).

9. 64x32-pixel monochrome display with the layout: (0,0) (63,0) (0,31) (63,31).

This is the physical game screen. It is a grid exactly 64 pixels wide and 32 pixels tall. Coordinate (0,0) is the very top-left pixel, and (63,31) is the very bottom-right pixel. "Monochrome" simply means there are no colors—a pixel is either completely ON (drawn) or completely OFF (empty).

10. Operation code

This is the raw 16-bit (2-byte) machine instruction (like 00 E0 or D0 14) that tells the CPU what action to take. The entire CPU cycle revolves around fetching an opcode, decoding what it means, and executing it.

## ROM
Acronym for Read-Only Memory, where the information is permanently burned into a physical microchip at the factory. The CPU can read from it as much as it wants, but it cannot easily write over it or erase it. When the power goes off, the information is perfectly safe.

In a emulator perspective, ROM is an instruction set that utilizes the console device to output a game. Basically the game itself that includes all the instructions, including graphics, sound, movement, memory, and more.

Inside a typical game ROM, there will be the instruction code and the game graphics. The bulk of the ROM is made up of the 16-bit opcodes we talked about earlier. These are the instructions that tell the game what to do. Unlike modern files that have a "header" at the top, CHIP-8 ROMs have absolutely zero metadata. The very first byte of the file is almost always the first half of the first opcode the game wants to run.

On the graphics side of the ROM, the basic 16-bit sprites are encoded into the emulator itself, while the specific game sprites are inside the ROMs. Since the simple system cannot read modern image extentions such as `.png` or `.jpg`, the clever way to create these sprites was to use a 5 byte binary representation of the object. As an example:
The character `F` will be represented as:
```
11110000
10000000
11110000
10000000
10000000
```
Where the numbers 1 represented that the pixel was on.

As a total example of a CHIP-8 game ROM file, it would look something like this:
```
ADDRESS |  HEX DATA  |  CATEGORY     |  WHAT IT ACTUALLY MEANS
---------|------------|---------------|----------------------------------
         |            |               |
 $200    |  00 E0     |  [GAME CODE]  |  Clear the screen
 $202    |  A2 0C     |  [GAME CODE]  |  Set Index to address $20C
 $204    |  60 1C     |  [GAME CODE]  |  Set X coordinate to 28
 $206    |  61 0E     |  [GAME CODE]  |  Set Y coordinate to 14
 $208    |  D0 14     |  [GAME CODE]  |  Draw 4 lines of graphics
 $20A    |  12 0A     |  [GAME CODE]  |  Loop forever (freeze)
         |            |               |
---------|------------|---------------|----------------------------------
         |            |               |
 $20C    |  F0        |  [GRAPHICS]   |  ████
 $20D    |  90        |  [GRAPHICS]   |  █  █
 $20E    |  90        |  [GRAPHICS]   |  █  █
 $20F    |  F0        |  [GRAPHICS]   |  ████
         |            |               |
```

# Development

### OOP Structure
Well reading from the CHIP-8 hardware description, wouldn't this be the perfect example for a object-oriented programming experience? That is exactly what we are going to do.

Our components can orient around the single object CHIP-8:
```cpp
#include <cstdint>

class Chip8 {
  public:
  	uint8_t registers[16]{};     // 16 8-bit registers
  	uint8_t memory[4096]{};      // 4kb memory
  	uint16_t index{};            // index register 16-bit
  	uint16_t pc{};               // program counter 16-bit
  	uint16_t stack[16]{};        // 16-level stack
  	uint8_t sp{};                // 8-bit stack pointer
  	uint8_t delayTimer{};        // 8-bit delay timer
  	uint8_t soundTimer{};        // 8-bit sound timer
  	uint8_t keypad[16]{};        // 16 input keys
  	uint32_t video[64 * 32]{};   // monochrome display memory(32-bit for SDL implementation)
  	uint16_t opcode;             // operation code of the current call
};
```
We will start our developemnt with this basis. 

### Font Sets
Explained on the graphics side of the hardware, we can initialize the 16 sprites for basic languages (0~9, A~F).
We will include these as a 8-bit to 80 long array such as:

```cpp
const unsigned int FONTSET_SIZE = 80;

uint8_t fontset[FONTSET_SIZE] = {
	0xF0, 0x90, 0x90, 0x90, 0xF0, // 0
	0x20, 0x60, 0x20, 0x20, 0x70, // 1
	0xF0, 0x10, 0xF0, 0x80, 0xF0, // 2
	0xF0, 0x10, 0xF0, 0x10, 0xF0, // 3
	0x90, 0x90, 0xF0, 0x10, 0x10, // 4
	0xF0, 0x80, 0xF0, 0x10, 0xF0, // 5
	0xF0, 0x80, 0xF0, 0x90, 0xF0, // 6
	0xF0, 0x10, 0x20, 0x40, 0x40, // 7
	0xF0, 0x90, 0xF0, 0x90, 0xF0, // 8
	0xF0, 0x90, 0xF0, 0x10, 0xF0, // 9
	0xF0, 0x90, 0xF0, 0x90, 0x90, // A
	0xE0, 0x90, 0xE0, 0x90, 0xE0, // B
	0xF0, 0x80, 0x80, 0x80, 0xF0, // C
	0xE0, 0x90, 0x90, 0x90, 0xE0, // D
};
```
The hardware will have the fontset loaded into the memory starting at 0x50, so we add that as well:
```cpp
const unsigned int FONTSET_START_ADDRESS = 0x50;
Chip8::Chip8() {
...
         for (int i = 0; i < FONTSET_SIZE; ++i) {
                  memory[FONTSET_START_ADDRESS + i] = fontset[i];
         }
}
```

### Loading a ROM
To load the instructions sets on a ROM, we must have the following:
- Get the size of the ROM
- Read the file
- Transfer the file contents to CHIP-8 memory (starting at 0x200)
- Put the program counter at the starting address (0x200)

Since we only need to read, we will use a standard ifstream to read the file. Of course the file should be read in binary, and since we want to get the size, we will move the file pointer to the end.
This will look something like:

```cpp
#include <fstream>

bool Chip8::load(const char *file_path) {
         std::ifstream file(file_path, std::ios::binary)

         if(!file) {
                  std::cerr << "Failed to open ROM" << std::endl;
                  return false;
         }
         
}
```
