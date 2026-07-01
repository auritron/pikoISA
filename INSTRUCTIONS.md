Instructions are written as it is, pseudo instructions are written with a `$` prefixed and resolved by the assembler.

(**Note: If an encoded instruction does not match the format of any of the implemented instructions, it raises an** `InvalidOpcode` **exception.)**

### Core Instructions

**Memory and registers -** 

- **`loadb`** - `loadb Rdst &Rsrc`
    - Load 1 byte (Quarterword) from RAM to Register
    - Sign extends after loading
    - Raises `InvalidAccess` if attempting to access out-of-bounds memory
- **`loadbu`** - `loadbu Rdst &Rsrc`
    - Load 1 byte unsigned from RAM to Register
    - Zero extends after loading
    - Raises `InvalidAccess` if attempting to access out-of-bounds memory
- **`sendb`** - `send Rsrc &Rdst`
    - Send lowest byte from Register to RAM
    - Raises `InvalidAccess` if attempting to access out-of-bounds memory
- **`$set`** - `$set Rdst Rsrc`
    - Sets the value of one register as another
    - Implemented as the following -
        
        ```nasm
        ; For when the second operand is a Register
        add Rdst Rsrc #0
        ```
        
- **`$seti`**  - `$set Rdst #IMM (32bit)`
    - Sets the value of one entire register as a 32 bit immediate operand
    - Implemented as the following -
        
        ```nasm
        ; For when the second operand is an Immediate
        setlhi Rdst (#IMM & 0xFFFF)
        setrhi Rdst (#IMM >> 16 & 0xFFFF)
        ; Above expressions are pseudocode representations resolved by the assembler
        ```
        
- **`setlhi`** - `setlhi Rdst #IMM (16bit)`
    - Sets the lower 16 bits of Rdst to an immediate value
- **`setuhi`** - `setuhi Rdst #IMM (16bit)`
    - Sets the upper 16 bits of Rdst to an immediate value
- **`sload`** - `sload Rdst Ssrc`
    - Load from Special Purpose Register Ssrc to GPR Rdst
    - Zero extended by default
- **`ssend`** - `ssend Rsrc Sdst`
    - Store the value of Rsrc into Special Purpose Register Sdst
    - Does not zero extend while loading into Sdst, although the upper bits are ignored if the computer only uses the lower bits of the respective register

**Arithmetic, logic, and shifting -**

- **`not`** - `not Rdst [Rsrc | #IMM (16bit)]`
    - Performs NOT of Rsrc and stores it in Rdst
    - If IMM, sign extends the immediate, performs NOT, and stores in Rdst
- **`or`** - `or Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Performs OR of Rsrc1 and Rsrc2 and stores it in Rdst
    - If IMM, sign extends the immediate, performs OR with Rsrc1, and stores in Rdst
- **`and`** - `and Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Performs AND of Rsrc1 and Rsrc2 and stores it in Rdst
    - If IMM, sign extends the immediate, performs AND with Rsrc1, and stores in Rdst
- **`xor`** - `xor Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Performs XOR of Rsrc1 and Rsrc2 and stores it in Rdst
    - If IMM, sign extends the immediate, performs XOR with Rsrc1, and stores in Rdst
- **`stl`** - `stl Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Performs signed LEFT SHIFT of Rsrc1 by the value stored in Rsrc2 or Immediate, and stores in Rdst
- **`$ustl`** - `$ustl Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Performs unsigned LEFT SHIFT of Rsrc1 by the value stored in Rsrc2 or Immediate, and stores in Rdst
    - As unsigned and signed left shift are the same, it is marked as a pseudo-instruction, implemented as -
        
        ```nasm
        ;For Registers
        stl Rdst Rsrc1 Rsrc2
        ;For Immediates
        stl Rdst Rsrc #IMM
        ```
        
- **`str`** - `str Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Performs signed RIGHT SHIFT of Rsrc1 by the value stored in Rsrc2 or Immediate, and stores in Rdst
    - Automatically performs sign extension after shifting
- **`ustr`** - `ustr Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Performs unsigned RIGHT SHIFT of Rsrc1 by the value stored in Rsrc2 or Immediate, and stores in Rdst
    - Automatically performs zero extension after shifting
- **`$ustr`** - `$ustr Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Pseudo-instruction that serves as an alias for `ustr`
    - Implemented as -
    
    ```nasm
    ;For Registers
    ustr Rdst Rsrc1 Rsrc2
    ;For Immediates
    ustr Rdst Rsrc #IMM
    ```
    
- **`add`** - `add Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Add the values of Rsrc1 and Rsrc2 and stores the value in Rdst
    - If IMM, adds the values of Rsrc1 and #IMM, and stores the value in Rdst
- **`sub`** - `sub Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Subtracts the value of Rsrc2 from Rsrc1, and stores the value in Rdst
    - If IMM, subtracts #IMM from Rsrc1, and stores the value in Rdst
- **`$neg`** - `$neg Rdst [Rsrc | #IMM (16bit)]`
    - Negates the value of Rsrc or #IMM and stores it in Rdst
    - Implemented as -
        
        ```nasm
        ;For Registers
        setlhi Rdst #0
        setrhi Rdst #0
        sub Rdst Rdst Rsrc
        ;For Immediates
        setlhi (0 - #IMM) ;Assembler automatically flips sign and extends #IMM
        setrhi (0 - #IMM)
        ```
        

**Jumping and branching -**

- **`cmp`** - `cmp Rx Ry`
    - Compares Rx’s and Ry’s values
    - Tracks Overflow (V) flag
    - Sets flags accordingly -
        - If Rx == Ry, Z = 1, else Z = 0
        - If Rx < Ry, Z = 0 and N = 1
        - If Rx > Ry, Z = 0 and N = 0
- **`goto`** - `goto [@LABEL | &Rlbl]`
    - Unconditional jump to given address
- **`jmp`** - `jmp #IMM (4bit) [@LABEL | &Rlbl]`
    - Conditional jump to given address, given by the code as #IMM
    - If the #IMM value is invalid, CPU raises an `InvalidOpcodeException`
    - List of Jump Codes are given below -
    
    | Jump Code | Opcode | Condition |
    | --- | --- | --- |
    | `#0x00` | `$jeq` | Jump if Z == 1 |
    | `#0x01` | `$jne` | Jump if Z == 0 |
    | `#0x02` | `$jcy` | Jump if C == 1 |
    | `#0x03` | `$jov` | Jump if V == 1 |
    | `#0x04` | `$jlt` | Jump if N ≠ V |
    | `#0x05` | `$jle`  | Jump if Z == 1 OR N ≠ V |
    | `#0x06` | `$jgt` | Jump if Z == 0 AND N == V |
    | `#0x07` | `$jge` | Jump if N == V |
    | `#0x08` | `$jltu` | Jump if C == 0 |
    | `#0x09` | `$jleu`  | Jump if C == 0 OR Z == 1 |
    | `#0x0A` | `$jgtu` | Jump if C == 1 AND Z == 0 |
    | `#0x0B` | `$jgeu` | Jump if C == 1 |
- **`$cal`** - `$cal [@LABEL | &Rlbl]`
    - Jumps unconditionally to given address, and pushes the current address it to the stack
    - Implemented as -
        
        ```nasm
        ; $cal &Rlbl
        sload RTMP SPC
        add RTMP RTMP #4
        sub RSP RSP #4    ;Push to stack
        send RTMP &RSP
        goto &Rlbl
        ```
        
- **`$calc`** - `$calc #IMM (4bit) [@LABEL | &Rlbl]`
    - Jumps conditionally to a given address, given by the code as #IMM, and pushes the current address to the stack
    - If the #IMM value is invalid, CPU raises an `InvalidOpcodeException`
    - Implemented as -
        
        ```nasm
        ; $calc #IMM &Rlbl
        sload RTMP SPC
        add RTMP RTMP #4
        sub RSP RSP #4    ;Push to stack
        send RTMP &RSP
        jmp #IMM &Rlbl
        ```
        
- **`$ret`** - `$ret`
    - Returns to the top address saved in the stack (as previously pushed by `$cal` or `$calc`
    - Implemented as -
        
        ```nasm
        ; $ret
        load RTMP &RSP
        add RSP RSP #4
        ssend RTMP SPC
        ```
        

**Stack manipulation -**

- **`$push`** - `$push Rsrc`
    - Pushes a value to the stack as stored in Rsrc
    - Decrements the program counter by 4 (as register size is 4 bytes) and then moves the value to the address in the stack
    - Implemented as -
        
        ```nasm
        ; $push Rsrc
        sub RSP RSP #4
        send Rsrc &RSP
        ```
        
- **`$pop`** - `$pop Rdst`
    - Pops the current value at the top of the stack (as indicated by `RSP`)
    - Increments the program counter by 4, and then moves the value to the register given by the instruction
    - Implemented as -
        
        ```nasm
        ; $pop Rdst
        load Rdst &RSP
        add RSP RSP #4
        ```
        

**Halt CPU Execution Flow -**

- **`sleep`** - `sleep`
    - Halts CPU execution, puts it to sleep until there is an external hardware interrupt (to be specified later)

### Core Extended Instructions  (CoreX)-

**Memory and Registers -**

- **`load2b`** - `load2b Rdst &Rsrc`
    - Loads byte at Rsrc, along with the byte at the address right after Rsrc into the lower 16 bits of Rdst, and performs *sign extension* for the upper bits
    - Raises `InvalidAccess` if attempting to access out-of-bounds memory, or `UnalignedAccess` if [Rsrc…Rsrc + 1] is not aligned to a 2-byte boundary
- **`load2bu`** - `load2bu Rdst &Rsrc`
    - Loads byte at Rsrc, along with the byte at the address right after Rsrc into the lower 16 bits of Rdst, and performs *zero extension* for the upper bits
    - Raises `InvalidAccess` if attempting to access out-of-bounds memory, or `UnalignedAccess` if [Rsrc…Rsrc + 1] is not aligned to a 2-byte boundary
- **`load4b`** - `load4b Rdst &Rsrc`
    - Loads byte at Rsrc, along with the next 3 bytes at the addresses right after Rsrc into Rdst
    - Raises `InvalidAccess` if attempting to access out-of-bounds memory, or `UnalignedAccess` if [Rsrc…Rsrc + 3] is not aligned to a 4-byte boundary
- **`send2b`** - `send2b Rsrc &Rsrc`
    - Send lowest 2 bytes from Register to RAM
    - Raises `InvalidAccess` if attempting to access out-of-bounds memory, , or `UnalignedAccess` if [Rdst…Rdst + 1] is not aligned to a 2-byte boundary
- **`send4b`** - `send4b Rsrc &Rsrc`
    - Send the 4-byte value from Register to RAM
    - Raises `InvalidAccess` if attempting to access out-of-bounds memory, or `UnalignedAccess` if [Rdst…Rdst + 3 is not aligned to a 4-byte boundary
- **`$swap`** - `$swap Rx Ry`
    - Swaps the values of Rx and Ry
    - Implemented as -
        
        ```nasm
        ; $swap Rx Ry
        add RTMP Rx #0
        add Rx Ry #0
        add Ry RTMP #0
        ```
        

**Arithmetic, logic, & shifting -**

- **`$nor`** - `$nor Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Calculates NOR of Rsrc1 and Rsrc2, and stores the value in Rdst
    - Implemented as -
    
    ```nasm
    ; $nor Rdst Rsrc1 [Rsrc2 | #IMM (16bit)
    or Rdst Rsrc1 Rsrc2
    not Rdst Rdst
    ```
    
- **`$nand`** - `$nand Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Calculates NAND of Rsrc1 and Rsrc2, and stores the value in Rdst
    - Implemented as -
    
    ```nasm
    ; $nand Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]
    and Rdst Rsrc1 Rsrc2
    not Rdst Rdst
    ```
    
- **`$xnor`** - `$xnor Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Calculates XNOR of Rsrc1 and Rsrc2, and stores the value in Rdst
    - Implemented as -
    
    ```nasm
    ; $xnor Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]
    xor Rdst Rsrc1 Rsrc2
    not Rdst Rdst
    ```
    
- **`rtl`** - `rtl Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Rotates the value of Rdst to the left by the value in Rsrc2 or #IMM, and stores the value in Rdst, so that they wrap around from the right
    - Performs the operation regardless of sign
- **`rtr`** - `rtr Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Rotates the value of Rdst to the right by the value in Rsrc2 or #IMM, and stores the value in Rdst, so that they wrap around from the left
    - Performs the operation regardless of sign
- **`$inc`** - `$inc Rdst Rsrc`
    - Increments the value of Rsrc by 1 and stores it in Rdst
    - Can cause overflow back to 0
    - Implemented as -
        
        ```nasm
        ; $inc Rdst Rsrc
        add Rdst Rsrc #1
        ```
        
- **`$dec`** - `$dec Rdst Rsrc`
    - Decrements the value of Rsrc by 1 and stores it in Rdst
    - Can cause underflow back to 0
    - Implemented as -
        
        ```nasm
        ; $dec Rdst Rsrc
        sub Rdst Rsrc #1
        ```
        
- **`mul`** - `mul Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Multiples the value of Rsrc1 with Rsrc2 or #IMM, stores the result in Rdst
    - If there is an overflow, the V flag is set to 1 and the overflow bits are stored in RTMP
    - Performs sign extension, and does not modify the RTMP register if there is no overflow
- **`umul`** - `umul Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Multiples the value of Rsrc1 with Rsrc2 or #IMM, stores the result in Rdst
    - If there is an overflow, the V flag is set to 1 and the overflow bits are stored in RTMP
    - Performs sign extension, and does not modify the RTMP register if there is no overflow
- **`div`** - `div Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Divides the value of Rsrc1 by Rsrc2 or #IMM, stores the quotient in Rdst
    - Raises `ZeroDivisionException` if the second operand is zero
    - Performs automatic sign extension
- **`udiv`** - `udiv Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Divides the value of Rsrc1 by Rsrc2 or #IMM, stores the quotient in Rdst
    - Raises `ZeroDivisionException` if the second operand is zero
    - Performs zero extension
- **`mod`** - `mod Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Divides the value of Rsrc1 by Rsrc2 or #IMM, stores the remainder in Rdst
    - Raises `ZeroDivisionException` if the second operand is zero
    - Performs sign extension
- **`umod`** - `umod Rdst Rsrc1 [Rsrc2 | #IMM (16bit)]`
    - Divides the value of Rsrc1 by Rsrc2 or #IMM, stores the remainder in Rdst
    - Raises `ZeroDivisionException` if the second operand is zero
    - Performs zero extension

**Jumping and branching -**

- **`$jeq`** — `$jeq [@LABEL | &Rlbl]`
    - Jumps to the given address if the zero flag is set.
    - Implemented as - `$jmp #0 &Rlbl`
- **`$jne`** — `$jne [@LABEL | &Rlbl]`
    - Jumps to the given address if the zero flag is not set.
    - Implemented as - `$jmp #1 &Rlbl`
- **`$jcy`** — `$jcy [@LABEL | &Rlbl]`
    - Jumps to the given address if the carry flag is set.
    - Implemented as - `$jmp #2 &Rlbl`
- **`$jov`** — `$jov [@LABEL | &Rlbl]`
    - Jumps to the given address if the overflow flag is set.
    - Implemented as - `$jmp #3 &Rlbl`
- **`$jgt`** — `$jgt [@LABEL | &Rlbl]`
    - Jumps to the given address if both the zero flag is not set and the negative flag equals the overflow flag (signed greater than).
    - Implemented as - `$jmp #4 &Rlbl`
- **`$jge`** — `$jge [@LABEL | &Rlbl]`
    - Jumps to the given address if the negative flag equals the overflow flag (signed greater than or equal to).
    - Implemented as - `$jmp #5 &Rlbl`
- **`$jlt`** — `$jlt [@LABEL | &Rlbl]`
    - Jumps to the given address if the negative flag does not equal the overflow flag (signed less than).
    - Implemented as - `$jmp #6 &Rlbl`
- **`$jle`** — `$jle [@LABEL | &Rlbl]`
    - Jumps to the given address if either the zero flag is set or the negative flag does not equal the overflow flag (signed less than or equal to).
    - Implemented as - `$jmp #7 &Rlbl`
- **`$jgtu`** — `$jgtu [@LABEL | &Rlbl]`
    - Jumps to the given address if both the zero flag is not set and the carry flag is set (unsigned greater than / no borrow).
    - Implemented as - `$jmp #8 &Rlbl`
- **`$jgeu`** — `$jgeu [@LABEL | &Rlbl]`
    - Jumps to the given address if the carry flag is set (unsigned greater than or equal to / no borrow).
    - Implemented as - `$jmp #9 &Rlbl`
- **`$jltu`** — `$jltu [@LABEL | &Rlbl]`
    - Jumps to the given address if the carry flag is not set (unsigned less than / borrow occurred).
    - Implemented as - `$jmp #10 &Rlbl`
- **`$jleu`** — `$jleu [@LABEL | &Rlbl]`
    - Jumps to the given address if either the zero flag is set or the carry flag is not set (unsigned less than or equal to / borrow occurred).
    - Implemented as - `$jmp #11 &Rlbl`

**System -**

- **`sys`** - `sys`
- **`sysret`**  - `sysret`

### Floating Point Instructions (pkF) -

**Memory and Registers -**

- **`fload`** - `floadb Fdst, &Rsrc`
- **`fsend`** - `fsend Fsrc, &Rdst`