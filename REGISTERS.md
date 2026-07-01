**The list of 16 GPRs are as follows -**

- `RSP` - Stack Pointer
- `RBP` - Base Pointer (Used to temporarily store pointer values and create stack frames)
- `RTMP` - Temporary Register (Utilized by pseudo-instructions and to store overflow bits)
- `RRX0` - Reserved Register 1 (Reserved for extensibility and future support)
- `RRX1` - Reserved Register 2 (Reserved for extensibility and future support)
- `RRTN`  - Return Register (Store return values for functions, similar to accumulator)
    
    *pikoISA* (pk-32) has 16 general purpose registers (GPRs), and few other special purpose registers (SPRs). Out of the 16 GPRs, the first 5 (`RSP` , `RBP` , `RTMP` , `RRX0` , and `RRX1` ) **may be used by an instruction and overwrite the data in them,** hence making them **destructive.** However, the *piko* Architecture guarantees that the remaining 11 GPRs will NOT be used by any instruction, and are **safe** to use without the risk of overwriting or destruction. Out of these, three GPRs (`RRTN`, `RARG0`, and `RARG1` ) are designated by convention, rather than for a specific purpose. All GPRs start with the prefix `R` and all SPRs start with the prefix `S` . GPRs may be called by most instructions that use registers, while SPRs can only be directly modified by the CPU or very specific instructions. SPRs are all 32 bits in length, which gives them compatibility to be loaded into GPRs.
    
- `RARG0` - Argument Register 1 (Store first function or instruction argument)
- `RARG1` - Argument Register 2 (Store second function or instruction argument)
- `R8` - True General Purpose Register 8
- `R9` - True General Purpose Register 9
- `R10` - True General Purpose Register 10
- `R11` - True General Purpose Register 11
- `R12` - True General Purpose Register 12
- `R13` - True General Purpose Register 13
- `R14` - True General Purpose Register 14
- `R15` - True General Purpose Register 15

**The list of SPRs are as follows -**

- `SPC` - Program Counter
- `SFLAGS` - Flag register (contains NZCV flags, i.e. Negative, Zero, Carry, Overflow flags)
    - Format - `| 32 . . . . . . . UNUSED BITS . . . . . . . 4 | V | C | N | Z |`

**With the pkF (Floating point) extension,** *pikoISA* specifies the existence of 8 more floating point registers, all of which are 32-bit, all of which are true general purpose registers.

**The list of FPRs are as follows -**

- `F0` - Floating Point Register 0
- `F1` - Floating Point Register 1
- `F2` - Floating Point Register 2
- `F3` - Floating Point Register 3
- `F4` - Floating Point Register 4
- `F5` - Floating Point Register 5
- `F6` - Floating Point Register 6
- `F7` - Floating Point Register 7