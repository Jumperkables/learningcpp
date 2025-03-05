## Registers and Pointers
Before i go on, I want to cover typical registers and pointers used program execution even though its a bit out of the scope of C++.

#### What are registers?
- Extremely fast bits of countably small memory used directly by the CPU
- They are standardised by ISA (instruction set architecture)
- **Each register is 16 positions of Hex i.e. 64bits long or 8 bytes**
    * Notably, the size of memory addresses in 64 bit architecture
Lets start by looking at registers in GDB:
![registers](gdb_registers.png)

#### Are registers hard locations or are they abstraction
- **They are NOT an abstraction they're on the CPU physically**
- Well hold, my computer is running many programs, while i'm sat in my breakpoint surely these registers are just hogged by my C++ program?
    * Correct, I already guessed the answer t my own question here, the OS context switches these values out and in when it's C++'s turn on the CPU

#### Register Types:
| **Register** | Meaning |
| -------- | -------- |
| `rax`,`rbx`, `rcx`,... | General purpose - Often for temp storage, return values, loop counters |