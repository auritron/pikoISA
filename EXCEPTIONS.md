*pikoISA’s* specification lists exceptions that a CPU using this architecture must implement, with both the core instructions and the implemented Exceptions.

## List of Exceptions -

| Vector Number | Exception Name | Description | Extension |
| :--- | :--- | :--- | :--- |
| `0x00` | `InvalidOpcode` | Raised by the processor if an invalid opcode or opcode format is passed | Core |
| `0x01` | `InvalidAccess` | Raised by the processor if the instruction tries to access an address that's either out of bounds or doesn't exist | Core |
| `0x02` | `UnalignedAccess` | Multi-byte access failure or alignment error | CoreX |
| `0x03` | `ZeroDivsion` | Divide by zero, in `div`, `udiv`, `mod`, or `umod` instructions, if the second operand is zero. | CoreX |
