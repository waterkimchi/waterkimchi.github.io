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

# CHIP-8
In the late 1970s, microcomputers had tiny amounts of RAM (often just 2KB to 4KB) and slow processors. Writing a game required writing raw assembly language or machine code specific to that computer's exact processor chip. This was incredibly tedious and difficult.  

Joseph Weisbecker wrote a tiny 512-byte interpreter program (a virtual machine) that sat in the computer's memory. This interpreter ran a much simpler, universal language he designed: CHIP-8. Instead of dealing with complex hardware code, hobbyists could write games using CHIP-8's straightforward commands, and the interpreter would translate them on the fly.

Here are the general specs of the CHIP-8:

1. 4,096 bytes (4KB) of addressable RAM, indexed from 0x000 to 0xFFF.

- 0x000 to 0x1FF: Originally reserved for the interpreter itself. In modern emulators, this space is typically empty and used to store font data.
- 0x200 to 0xFFF: Where the actual game data (ROM) is loaded and executed.

2. 16 general purpose 8-bit registers (V0 through VF).

A register is basically a location on a CPU for storage. All instructions and operations must be done within the resgisters. Since it is 8-bit, that means each register is only able to hold any value from 0x00 to 0xFF. The last register (VF) is a "flag" register used to handle math carries and graphics collisions, basically an error flag.

3. Index 16-bit register (I) to store memory addresses.

4. Program counter (16-bit pointer) that tracks the memory address of the current instruction.

5. Keyboard

The orignially CHIP-8 machine responded to the key map of: 
1 2 3 C
4 5 6 D
7 8 9 E
A 0 B F
This layout must be mapped into various other configurations to fit the keyboards of today's platforms.

6. 64x32-pixel monochrome display.

The layout was:
(0,0)         (63,0)

(0,31)        (63,31)
