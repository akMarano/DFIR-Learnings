# ASSEMBLY LANGUAGE CHEAT SHEET
### 32-bit x86 Assembly — Intel Syntax

A reference guide covering registers, memory, stack, functions, instructions, comparisons, jumps, loops, and assembly problem-solving.

---

# 1. BASIC COMPUTER TERMINOLOGY

| Term | Meaning |
|---|---|
| Assembly language | Human-readable instructions for the processor |
| Machine language | Binary instructions executed by the processor |
| CPU | Central Processing Unit; executes instructions |
| RAM | Temporary memory used by programs |
| Bit | A single binary digit (0 or 1) |
| Byte | 8 bits |
| Word | 2 bytes (16 bits) |
| DWORD | Double word = 4 bytes (32 bits) |
| QWORD | Quad word = 8 bytes (64 bits) |
| Binary | Base-2 number system |
| Decimal | Base-10 number system |
| Hexadecimal | Base-16 number system |
| `0x` | Prefix indicating a hexadecimal number |

## Number system conversions

| Number system | Digits used |
|---|---|
| Binary | 0, 1 |
| Decimal | 0–9 |
| Hexadecimal | 0–9, A–F |

Hexadecimal letter values:

| Hex | Decimal |
|---|---:|
| A | 10 |
| B | 11 |
| C | 12 |
| D | 13 |
| E | 14 |
| F | 15 |

### Hexadecimal examples

| Hexadecimal | Decimal |
|---|---:|
| `0x1` | 1 |
| `0x4` | 4 |
| `0x8` | 8 |
| `0x10` | 16 |
| `0x17` | 23 |
| `0x94` | 148 |
| `0x14b` | 331 |
| `0x2fa` | 762 |
| `0x31e` | 798 |
| `0x38f0` | 14576 |

**IMPORTANT:** `17` and `0x17` are different.

- `17` = decimal seventeen.
- `0x17` = hexadecimal seventeen = decimal twenty-three.

---

# 2. REGISTERS

Registers are small storage locations built into the CPU.

Think of them as temporary containers used to store numbers, addresses, and calculation results.

## Register sizes

The register prefixes help identify their sizes:

| Prefix | Size | Example |
|---|---:|---|
| No prefix | 16 bits | AX, BX, CX, DX |
| E prefix | 32 bits | EAX, EBX, ECX, EDX |
| R prefix | 64 bits | RAX, RBX, RCX, RDX |

Remember:

- AX = 16 bits.
- EAX = 32 bits.
- RAX = 64 bits.

These are related portions of the same architectural register in x86-64.

## Smaller portions of registers

The smaller register names let you access only specific portions of a larger register.

### Example: EAX

EAX is 32 bits.

| Register | Size | What it accesses |
|---|---:|---|
| RAX | 64 bits | Entire 64-bit register |
| EAX | 32 bits | Lower 32 bits of RAX |
| AX | 16 bits | Lower 16 bits of EAX |
| AH | 8 bits | Bits 8–15 of AX |
| AL | 8 bits | Bits 0–7 of AX |

Visual representation:

```text
              EAX (32 bits)
     ┌─────────────────────────┐
     │          EAX            │
     └─────────────────────────┘

              AX (16 bits)
     ┌─────────────┬───────────┐
     │     AH      │    AL     │
     │   8 bits    │  8 bits   │
     └─────────────┴───────────┘
        Bits 8–15     Bits 0–7
```

Remember:

- AH = upper 8 bits of AX.
- AL = lower 8 bits of AX.
- AX = AH + AL.
- AX is the lower 16 bits of EAX.
- EAX is the lower 32 bits of RAX.

**Important:** AH is NOT the upper 8 bits of the entire EAX. It is the upper 8 bits of AX.

### Smaller portions of other registers

| 64-bit | 32-bit | 16-bit | Lower 8-bit | Upper 8-bit |
|---|---|---|---|---|
| RAX | EAX | AX | AL | AH |
| RBX | EBX | BX | BL | BH |
| RCX | ECX | CX | CL | CH |
| RDX | EDX | DX | DL | DH |

For other registers, such as RSI and RDI, the 8-bit portions in 64-bit mode are SIL and DIL.

### Important register behavior

In 64-bit mode:

```asm
mov eax, 10
```

Writing to EAX automatically clears the upper 32 bits of RAX.

However, writing to AX, AH, or AL does not automatically clear the other portions.

---

# 3. IMPORTANT REGISTERS TO REMEMBER

| Register | Purpose | Easy way to remember |
|---|---|---|
| EAX | Calculations and return values | Result container |
| EBX | General-purpose register | Working storage |
| ECX | General-purpose, sometimes a loop counter | Counter |
| EDX | General-purpose, also used in arithmetic | Working storage |
| EBP | Base pointer | Stack-frame reference |
| ESP | Stack pointer | Top of stack |
| ESI | Source index | Source pointer |
| EDI | Destination index | Destination pointer |
| EFLAGS | Condition flags | Status indicators |
| EIP | Instruction pointer | Next instruction address |

### The three most important registers in your exercises

**EAX:** Holds calculations and the return value.

**EBP:** Provides a stable reference point for the current function's stack frame.

**ESP:** Tracks the current top of the stack.

---

# 4. MEMORY AND MEMORY ADDRESSES

This is one of the most important concepts in assembly.

There are TWO different things you need to understand:

1. Memory address = WHERE something is stored.
2. Memory value = WHAT is stored there.

Example:

| Address | Stored value |
|---|---:|
| 1000 | 2000 |
| 1004 | 50 |
| 1008 | 798 |

Address 1008 contains the value 798.

**The address is NOT the value.**

Think of it like a house:

- Address = location of the house.
- Value = what is inside the house.

## Square bracket notation `[ ]`

Square brackets mean:

**Access the value stored at a memory address.**

Example:

```asm
mov eax, [ebp+8]
```

This means:

1. Calculate EBP + 8.
2. Go to that memory address.
3. Read the value stored there.
4. Copy the value into EAX.

### Difference between these two:

```asm
ebp + 8
```

Calculates a memory address.

```asm
[ebp + 8]
```

Accesses the value stored at that memory address.

## What does DWORD PTR mean?

Example:

```asm
DWORD PTR [ebp+8]
```

| Part | Meaning |
|---|---|
| DWORD | 4 bytes |
| PTR | Pointer/reference to memory |
| `[ ]` | Access memory contents |
| `ebp+8` | Calculate the address |

In simple terms:

Read a 4-byte value from memory at EBP + 8.

## Common addressing modes

| Example | Meaning |
|---|---|
| `0x1000` | Direct address |
| `[eax]` | Access memory at the address stored in EAX |
| `[ebp-4]` | Access memory at EBP minus 4 |
| `[ebp+8]` | Access memory at EBP plus 8 |
| `[eax+ecx]` | Access memory at EAX + ECX |
| `[eax+ecx*4]` | Access memory at EAX + ECX × 4 |

---

# 5. THE STACK

The stack is a section of memory used to store information associated with function calls.

It follows the principle:

**LIFO = Last In, First Out.**

The last item pushed onto the stack is the first item popped off.

## Stack direction

The stack grows DOWNWARD, toward smaller memory addresses.

```text
Higher memory addresses
        |
        v
      1000
       996
       992
       988
        |
        v
Lower memory addresses

Stack grows downward.
```

### Two important rules

- PUSH decreases ESP.
- POP increases ESP.

## How many bytes does PUSH take?

The amount depends on the architecture and operand size.

For normal pushes:

| Architecture / operand size | Bytes pushed | Stack pointer change |
|---|---:|---:|
| 16-bit push | 2 bytes | -2 |
| 32-bit push | 4 bytes | -4 |
| 64-bit push | 8 bytes | -8 |

For your 32-bit exercises:

**A normal PUSH takes 4 bytes.**

Examples:

```asm
push eax
push ebp
push ebx
```

Each decreases ESP by 4.

The number being pushed does NOT determine how much ESP decreases.

For example:

```asm
push 0x20
```

In 32-bit mode, a normal push of this immediate uses a 4-byte stack slot, even though `0x20` equals 32.

## PUSH

```asm
push eax
```

Places the value of EAX onto the stack.

Conceptually:

```text
ESP = ESP - 4
Memory[ESP] = EAX
```

## POP

```asm
pop eax
```

Takes the topmost stack value and puts it into EAX.

Conceptually:

```text
EAX = Memory[ESP]
ESP = ESP + 4
```

---

# 6. EBP AND ESP

These two registers are closely related but serve different purposes.

| Register | Purpose |
|---|---|
| ESP | Points to the current top of the stack |
| EBP | Provides a stable reference point for the current function's stack frame |

## Why do we save EBP?

Imagine Function A calls Function B.

Function A already has its own EBP, which it uses to locate its variables.

Function B also needs its own EBP.

But there is only one EBP register.

Therefore, Function B saves the previous EBP before using the register for its own stack frame.

At the beginning:

```asm
push ebp
mov ebp, esp
```

At the end:

```asm
pop ebp
ret
```

The first instruction saves the old EBP.

The last two restore the old EBP and return to the caller.

**Main reason:** We save EBP so that the calling function can still access its own stack frame after the called function finishes.

## Why does the stack diagram have EBP + 8?

In the conventional 32-bit stack layout used in your exercises:

| Memory location | Contains |
|---|---|
| `[ebp]` | Saved EBP |
| `[ebp+4]` | Return address |
| `[ebp+8]` | First argument |
| `[ebp+12]` | Second argument |
| `[ebp+16]` | Third argument |

Why is the first argument at EBP + 8?

Because:

- Saved EBP takes 4 bytes.
- Return address takes 4 bytes.

Therefore:

```text
4 + 4 = 8 bytes
```

This is how the function knows where to find the first argument.

These positions depend on the calling convention and stack-frame layout.

---

# 7. FUNCTION ARGUMENTS AND LOCAL VARIABLES

An argument is information supplied to a function by whoever calls it.

A local variable is storage used by the function itself.

For the conventional stack-frame layout in your examples:

| Location | Meaning |
|---|---|
| `[ebp+8]` | First argument |
| `[ebp+12]` | Second argument |
| `[ebp+16]` | Third argument |
| `[ebp-4]` | Local variable |
| `[ebp-8]` | Another local variable |

Remember:

- Positive EBP offsets commonly refer to function arguments.
- Negative EBP offsets commonly refer to local variables.

## Example

```c
asm2(222, 20);
```

Under the 32-bit stack-based convention used in your exercises, arguments are pushed RIGHT TO LEFT:

```asm
push 20
push 222
call asm2
```

After the function executes:

```asm
push ebp
mov ebp, esp
```

The arguments are located at:

```text
[ebp+8]  = 222 (first argument)
[ebp+12] = 20  (second argument)
```

---

# 8. HOW MUCH STACK SPACE IS RESERVED?

This is different from PUSH.

Consider:

```asm
sub esp, 0x10
```

Convert hexadecimal:

```text
0x10 = 16
```

This means:

```text
ESP = ESP - 16
```

The function reserves 16 bytes of local stack space.

## VERY IMPORTANT: PUSH versus SUB ESP

| Instruction | What happens |
|---|---|
| `push 0x10` | Pushes the value 16; ESP decreases by 4 in 32-bit mode |
| `sub esp, 0x10` | Subtracts 16 from ESP; reserves 16 bytes |

These instructions do different things.

## Data sizes

| Term | Size |
|---|---:|
| BYTE | 1 byte |
| WORD | 2 bytes |
| DWORD | 4 bytes |
| QWORD | 8 bytes |

Example:

```asm
mov DWORD PTR [ebp-4], eax
```

Stores 4 bytes.

```asm
mov QWORD PTR [rbp-8], rax
```

Stores 8 bytes.

**Remember:** DWORD and QWORD indicate the memory operation's data width. They do not automatically tell you how much ESP decreases.

---

# 9. FUNCTION SETUP AND CLEANUP

## Function prologue

The prologue prepares the function's stack frame.

```asm
push ebp
mov ebp, esp
```

Meaning:

1. Save the previous EBP.
2. Set up the new EBP reference point.

Sometimes you will also see:

```asm
sub esp, 0x10
```

This reserves 16 bytes of local stack space.

## Function epilogue

The epilogue cleans up the current function's stack frame.

Two common forms:

### Form 1

```asm
pop ebp
ret
```

### Form 2

```asm
leave
ret
```

`leave` performs the equivalent of:

```asm
mov esp, ebp
pop ebp
```

| Instruction | Meaning |
|---|---|
| `push ebp` | Save previous EBP |
| `mov ebp, esp` | Establish stack frame |
| `sub esp, number` | Reserve local stack space |
| `pop ebp` | Restore previous EBP |
| `leave` | Reset ESP and restore EBP |
| `ret` | Return to the caller |

---

# 10. ASSEMBLY INSTRUCTIONS

Your examples use **Intel syntax**.

In Intel syntax:

```asm
instruction DESTINATION, SOURCE
```

The destination comes first.

## Basic instructions

| Instruction | Meaning | Example |
|---|---|---|
| `mov` | Copy a value | `mov eax, 5` |
| `lea` | Calculate an address | `lea eax, [ebp-4]` |
| `add` | Addition | `add eax, 5` |
| `sub` | Subtraction | `sub eax, 5` |
| `imul` | Multiplication | `imul eax, 5` |
| `inc` | Increase by 1 | `inc eax` |
| `dec` | Decrease by 1 | `dec eax` |
| `neg` | Change the sign | `neg eax` |
| `and` | Bitwise AND | `and eax, ebx` |
| `or` | Bitwise OR | `or eax, ebx` |
| `xor` | Bitwise XOR | `xor eax, eax` |
| `not` | Bitwise NOT | `not eax` |
| `shl` | Shift bits left | `shl eax, 1` |
| `shr` | Logical shift right | `shr eax, 1` |
| `sar` | Arithmetic shift right | `sar eax, 1` |

## Understanding MOV

```asm
mov eax, 10
```

EAX = 10.

```asm
mov eax, ebx
```

Copy EBX into EAX.

```asm
mov eax, [ebp+8]
```

Read the value in memory at EBP + 8 and copy it to EAX.

```asm
mov [ebp-4], eax
```

Copy EAX into memory at EBP - 4.

## Understanding ADD and SUB

```asm
add eax, 5
```

EAX = EAX + 5.

```asm
sub eax, 5
```

EAX = EAX - 5.

## Understanding LEA

```asm
lea eax, [ebp-4]
```

LEA calculates the effective address without reading the memory contents.

If EBP = 1000:

```text
EAX = 1000 - 4
EAX = 996
```

It puts the address 996 into EAX, NOT the value stored at address 996.

---

# 11. COMPARISONS AND CONDITION FLAGS

## CMP

CMP compares two values.

Example:

```asm
cmp eax, 10
```

In Intel syntax, this conceptually calculates:

```text
EAX - 10
```

However, CMP does NOT store the subtraction result in EAX.

Instead, it updates the condition flags.

## Important flags

| Flag | Full name | Meaning |
|---|---|---|
| ZF | Zero Flag | Result was zero |
| SF | Sign Flag | Result was negative |
| OF | Overflow Flag | Signed overflow |
| CF | Carry Flag | Unsigned carry/borrow |

Think of these as indicators that tell the processor what happened during an operation.

### CMP examples

If EAX = 10:

```asm
cmp eax, 10
```

The result is zero.

ZF = 1.

If EAX = 20:

```asm
cmp eax, 10
```

The result is positive.

For this simple example, ZF = 0, SF = 0, OF = 0.

If EAX = 5:

```asm
cmp eax, 10
```

The result is negative.

For this simple example, ZF = 0, SF = 1, OF = 0.

---

# 12. CONDITIONAL JUMPS

These instructions determine where the processor goes next.

| Instruction | Meaning | Equivalent condition |
|---|---|---|
| `je` | Jump if equal | `a == b` |
| `jne` | Jump if not equal | `a != b` |
| `jg` | Jump if signed greater | `a > b` |
| `jge` | Jump if signed greater or equal | `a >= b` |
| `jl` | Jump if signed less | `a < b` |
| `jle` | Jump if signed less or equal | `a <= b` |
| `ja` | Jump if unsigned above | `a > b` |
| `jae` | Jump if unsigned above or equal | `a >= b` |
| `jb` | Jump if unsigned below | `a < b` |
| `jbe` | Jump if unsigned below or equal | `a <= b` |
| `jmp` | Always jump | Unconditional |

### Important conditions

**JE**

Jump if equal.

Checks whether ZF = 1.

**JNE**

Jump if not equal.

Checks whether ZF = 0.

**JG**

Jump if signed greater.

Condition:

```text
ZF = 0 AND SF = OF
```

**JLE**

Jump if signed less than or equal.

Condition:

```text
ZF = 1 OR SF != OF
```

**JMP**

Always jumps. Does not depend on condition flags.

### Remember

- JG = Greater.
- JNE = Not equal.
- JLE = Less than or equal.
- JMP = Always jump.

Also remember that JG/JLE are signed comparisons, whereas JA/JBE are unsigned comparisons.

---

# 13. PROGRAM FLOW AND INSTRUCTION OFFSETS

You may see something like:

```asm
<+18>: jmp 0x119c <asm2+31>
```

Understand each part:

| Part | Meaning |
|---|---|
| `<+18>` | Current instruction is 18 bytes from the start of the function |
| `jmp` | Jump instruction |
| `0x119c` | Destination memory address |
| `<asm2+31>` | Destination is 31 bytes from the start of asm2 |

`0x119c` and `asm2+31` identify the same destination.

**Offsets are measured in bytes, not instruction numbers.**

When a jump happens, the processor can skip instructions instead of executing every instruction in order.

Always follow the jump destination when tracing assembly.

---

# 14. LOOPS

A loop repeats instructions until a condition is no longer satisfied.

Example:

```asm
jmp check

loop:
    add eax, 1

check:
    cmp eax, 10
    jle loop
```

How it works:

1. Check a condition.
2. If TRUE, jump to the loop body.
3. Execute the instructions.
4. Check the condition again.
5. If FALSE, exit the loop.

## Important loop instructions

| Instruction | Meaning |
|---|---|
| `cmp` | Check a condition |
| `jle` | Repeat while less than or equal |
| `jg` | Jump if greater |
| `jmp` | Jump unconditionally |

### A useful distinction

If you see:

```asm
jmp check
```

before the loop body, the function checks the condition before the first iteration.

This is the pattern used in your asm2.

---

# 15. FUNCTION CALLS AND RETURN VALUES

A function is a block of instructions that performs a particular task.

Example:

```c
int add(int a, int b)
{
    return a + b;
}
```

| Term | Meaning |
|---|---|
| Function | A block of instructions |
| Argument | An input supplied to a function |
| Return value | The result given back |
| `call` | Call another function |
| `ret` | Return to the calling function |
| EAX | Common 32-bit integer return register |

## CALL

```asm
call asm1
```

CALL:

1. Pushes the return address onto the stack.
2. Transfers execution to asm1.

## RET

```asm
ret
```

RET takes the return address from the stack and returns execution to the calling function.

## Return value

In a typical 32-bit integer-returning function, EAX contains the return value.

Example:

```asm
mov eax, 775
ret
```

The function returns 775.

## Calling convention used in your exercises

Arguments are pushed RIGHT TO LEFT.

Example:

```c
asm2(222, 20);
```

Conceptually:

```asm
push 20
push 222
call asm2
```

After the usual function prologue:

```text
[ebp+8]  = 222 (first argument)
[ebp+12] = 20  (second argument)
```

Calling conventions can vary, so always check which convention the program uses.

---

# 16. YOUR ASM1 AND ASM2 EXAMPLES

## ASM1 — Conditional branching

### What it does

Receives one integer and adds or subtracts 23 depending on its value.

Equivalent C code:

```c
int asm1(int x)
{
    if (x > 762)
    {
        if (x == 798)
        {
            return x - 23;
        }
        else
        {
            return x + 23;
        }
    }
    else
    {
        if (x == 331)
        {
            return x + 23;
        }
        else
        {
            return x - 23;
        }
    }
}
```

### Example

```text
asm1(798)

798 > 762? YES
Jump to +37.

798 == 798? YES
JNE does not jump.

EAX = 798 - 23

EAX = 775
```

**Return value: 775**

## ASM2 — Looping

### What it does

Receives two integers.

- Increments the second argument by 1.
- Increments the first argument by 148.
- Repeats while the first argument is less than or equal to 14576.
- Returns the updated second argument.

Equivalent C code:

```c
int asm2(int x, int y)
{
    int local_y = y;
    int local_x = x;

    while (local_x <= 14576)
    {
        local_y++;
        local_x += 148;
    }

    return local_y;
}
```

### Example

```text
asm2(14280, 10)

Initial:
local_x = 14280
local_y = 10

Iteration 1:
local_x = 14428
local_y = 11

Iteration 2:
local_x = 14576
local_y = 12

Iteration 3:
local_x = 14724
local_y = 13

Check:
14724 <= 14576? NO

Return 13.
```

**Return value: 13**

---

# 17. STEP-BY-STEP METHOD FOR SOLVING ASSEMBLY EXERCISES

Whenever you receive a new assembly function, follow this order.

1. Identify whether the assembly is 32-bit or 64-bit.
2. Identify the registers being used.
3. Identify the function arguments.
4. Identify the function prologue.
5. Check whether SUB ESP reserves additional stack space.
6. Identify local variables using locations such as `[ebp-4]`.
7. Convert hexadecimal values into decimal.
8. Read MOV instructions to understand where values are copied.
9. Find CMP instructions to identify comparisons.
10. Identify conditional jumps such as JG, JNE, and JLE.
11. Follow every jump to its destination.
12. Determine whether the function has a loop.
13. Trace the calculations using actual input values.
14. Identify the return value, commonly stored in EAX.
15. Find the function epilogue and RET.
16. Translate the whole function into C or pseudocode.

---

# 18. COMMON MISTAKES TO AVOID

| Mistake | Correct understanding |
|---|---|
| Thinking `0x17` means 17 | It means decimal 23 |
| Thinking `[ebp+8]` means address 8 | It means memory at EBP + 8 |
| Thinking PUSH subtracts from the value | PUSH moves ESP downward |
| Thinking EBP and ESP are the same | EBP is a frame reference; ESP tracks the stack top |
| Thinking CMP stores its subtraction result | CMP updates flags only |
| Thinking JNE means jump if equal | It means jump if NOT equal |
| Thinking JMP checks a condition | JMP always jumps |
| Thinking EBP+8 is always the first argument | It depends on the calling convention and stack layout |
| Thinking reserved bytes equal argument bytes | Local stack space and arguments are separate |
| Reading instructions only top to bottom | Follow jump destinations |
| Thinking MOV calculates an address | LEA calculates an address |
| Thinking `sub esp, 0x10` is the same as `push 0x10` | SUB reserves 16 bytes; PUSH uses a 4-byte slot in 32-bit mode |

---

# 19. QUICK REFERENCE: ESSENTIAL TERMS

| Term | One-line meaning |
|---|---|
| EAX | Calculation and return register |
| EBP | Stack-frame reference |
| ESP | Stack pointer |
| EFLAGS | Condition flags |
| DWORD | 4 bytes |
| QWORD | 8 bytes |
| `[ ]` | Access memory contents |
| PUSH | Put a value onto the stack |
| POP | Remove a value from the stack |
| MOV | Copy a value |
| LEA | Calculate an address |
| ADD | Addition |
| SUB | Subtraction |
| CMP | Compare |
| JE | Jump if equal |
| JNE | Jump if not equal |
| JG | Jump if signed greater |
| JLE | Jump if signed less or equal |
| JMP | Always jump |
| CALL | Call a function |
| RET | Return from a function |
| LEAVE | Restore the stack frame |

---

# 20. THE FIVE MOST IMPORTANT CONCEPTS

If you remember nothing else, remember these five:

**1. REGISTERS**

Know what EAX, EBP, and ESP do.

**2. MEMORY**

Understand the difference between an address and the value stored at that address.

**3. STACK**

Understand why PUSH decreases ESP, how arguments are placed on the stack, and how EBP helps locate them.

**4. CONDITIONS**

Understand CMP, JG, JNE, JLE, and how they use condition flags.

**5. PROGRAM FLOW**

Follow jumps instead of blindly reading instructions from top to bottom.

**Final reminder:** Your asm1 and asm2 exercises are both 32-bit x86 assembly using Intel syntax. Some rules, especially argument locations and stack layouts, depend on the architecture and calling convention. Always identify those before analyzing a new function.

You've already successfully solved two assembly exercises. Keep practicing these concepts, and you'll become much more comfortable reading assembly independently.
