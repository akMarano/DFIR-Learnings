# Reverse Engineering

Reverse Engineering is the process of deconstructing software, hardware, or binaries without source code to understand how they work and find security flaws. Defenders use it to analyze malware, uncover vulnerabilities, and build better protections, while attackers may use similar methods to find exploits

### Initial Analysis
We can use Hex/Text editors to gather basic information regarding the file
```
xxd happydog.jpeg | head
00000000: ffd8 ffe0 0010 4a46 4946 0001 0100 0001  ......JFIF......
00000010: 0001 0000 ffdb 0043 0009 0607 1313 1215  .......C........
00000020: 1313 1215 1615 1518 1716 1518 1617 1715  ................
00000030: 1616 1715 1516 1615 1515 1818 1d28 2018  .............( .
00000040: 1a25 1b15 1521 3121 2529 2b2e 2e2e 171f  .%...!1!%)+.....
00000050: 3338 332d 3728 2d2e 2bff db00 4301 0a0a  383-7(-.+...C...
00000060: 0a0e 0d0e 1a10 101b 2d26 1f25 2d2d 2d2d  ........-&.%----
00000070: 2d2d 2f2d 2d2d 2d2b 2d2d 2d2d 2d2d 2d2d  --/----+--------
00000080: 2d2d 2d2d 2d2d 2d2d 2d2d 2d2d 2d2d 2d2d  ----------------
00000090: 2d2b 2b2d 2d2d 2d2d 2d2d 2d2d 2b2d ffc0  -++---------+-..
```
We can utilize the magic bytes of the file to determine its file type, in this case based on its headers `ffd8 ffe0 0010` and the JFIF text, that this file is a jpeg.

### Running Files
It is also important we monitor the changes a file makes when it is ran so we are able to analyze what it aims to do. For this we monitor the Network, Log Files, Windows Registry and etc.

For Windows Registry we have the following registers to check based on what we want to check.

**Windows Startup**
1. HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
2. HKLM\Software\Microsoft\Windows\CurrentVersion\Run
3. HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
4. HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnceEx
5. HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices
6. HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
7. HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
8. HKLM\SOFTWARE\Wow6432Node\Windows\CurrentVersion\Run

**Windows Login**
1. HKCU\Software\Microsoft\Windows\CurrentVersion\Run
2. HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
3. HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnceEx
4. HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices
5. HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
6. HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows\Run
7. HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows\Load
8. HKLM\SYSTEM\ControlSet***\Control\Session Manager\BootExecute

For **Linux**, since linux systems don't utilize registry like windows, we check the following
1. Check the cron configuration
2. Analyze processes running at startup
3. Analyze IPV6/IPV4 usage and configuration
4. Analyze USB detection and usage
5. Analyze X Server Usage
6. Monitor /boot
7. Use chroot jails
8. Analyze $PATH, aliases

### Binaries
Computers use binaries to store and process data but we often use hexadecimal to simplify how these binaries are represented. Binaries can also be easily represented as a list of ON and OFF switches which is why computers work with the binary system so well.

**Example:**
For Positive numbers, the most significant bit or the leftmost bit is 0, while the rest represent the actual number, below is an example of the number 87.

0101 0111<sub>2</sub> = 2<sup>6</sup> + 2<sup>4</sup> + 2<sup>2</sup> + 2<sup>1</sup> + 2<sup>0</sup> = 64 + 16 + 4 + 2 + 1 = 87<sub>10</sub>

For Negative numbers, we must first take the positive of the negative number, and then reverse the 1s and 0s, and then add 1 to the result and then set the most significant bit to 1.

-13 -> 00001101 (Binary of 13, the positive of -13)

00001101 -> 11110010 (Reverse the 1s and 0s)

11110010 -> 11110011 (Add 1)

11110011 -> 11110011 (Change the MSB to 1, in this case it was already 1)

### CPU Registers
An accesible location available to the CPU which has a small amount of fast storage, some registers have hardware functions and might be RO or WO. Computers load data from larger memories into the register where the arithmetic calculations take place, the results are then stored back to the main memory. Registers generally hold 8-64 bits or more.


**Instruction Pointer (IP)**

The register that contains the memory address where the next assembly instruction is that will be executed.

**General Purpose Registers (GPR)**
1. AX - Accumulator Register, storage for arithmetic operations
2. CX - Counter Register, shift/rotate and loops instructions
3. DX - Data Register, arithemtic and I/O operations, and general storage
4. BX - Base Register, stores the base address pointing to the data
5. SP - Stack Pointer, pointer to the top of the stack, the address held in SP is the RAM address where the last byte was stored by a stack operation. When data is added the SP increments, if data is taken then the SP decrements
6. BP - Base Pointer, local parameters passed via stack and more
7. SI - Source Index Register, pointer to a source in string/stream operations
9. DI - Destination Index Register, pointer to a destination in string/stream operations.

- 16 bits - The registers above are indicators of 16 bit registers
- 32 bits - The names above with a prefixed E like EAX are 32 bits
- 64 bits - the names above with a prefixed R like RAX are 64 bits

The most significant bits of the X registers (AX, BX, CX, and DX) can be accessed by smaller registers
- EAX is the full 32 bits
- AX is the lower 16 bits
- AL is the lower 8 bits
- AH is the bits 8-15

**Segment Registers **

Memory is divided into sections, the code, stack and data segments and more. Segment registers identify the starting location of these sections. Nowadays many applications and OS have these registers pointing to the same locations (generally zero) to disable them. Windows uses GS to store thread related data, while linux uses it to save per-CPU information.

1. SS - Pointer to the Stack
2. CS - Pointer to the Code
3. DS - Pointer to the Data
4. ES - Pointer to extra data
5. FS - Pointer to more extra data
6. GS - Pointer to even more extra data

Segmentation allowed the addressing of more than 64kb, The x86-64 architecture has dropped support for this technology

**FLAGS Register**

These registers are used to store the results of the operations and the state of the processor. This is a 32 bit register.
1. CF - on if an addition or subtraction borrowed a bit beyond the size of the register
2. PF - on if the number of set bits in the least significant byte is even, used with JUMP instructions
3. AF - on if borrow generated out of four least significant bits
4. ZF - on if the result of the operation is zero
5. SF - on if the result of the operation is negative
6. TF - on if the step-by-step execution for debugging is needed
7. IF - on if interrupts are enabled
8. DF - on if string operations will read memory backwards
9. OF - on if arithmetic overflow has happened
10. IOPL - I/O Privilege level of current process
11. NT - on if the current process is linked to the next one
12. RF - turns some debug exceptions off to allow debugging
13. VM - on to allow some real mode applications to run in protected mode
14. AC - on if alignment checking of memory references is done
15. VIF - on if VIP then OS will service deferred interrupts
16. VIP - on if an interrupt is pending
17. ID - on if support for CPUID instruction can be set

### Memory Addressing
The x86 architecture is little-endian, the multi-byte values are written least significant byte first

A3A2A1A0 is represented as A0A1A2A3
