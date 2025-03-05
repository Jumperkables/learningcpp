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

#### Why aren't there more registers?
- They are simply that expensive to have on the CPU. More die area etc.
- Adding more registers means more complex instructions
    * Right now, compilers know which registers they're getting. They know how to use them. Adding more complicates things
- Diminishing returns?
    * 16 of these registers are all thats needed for C++ and modern code
    * Most attempts to improve have found improvements not worth the cost

#### Hold on, only 16?! Why are the other registers unused? Which 16?
| **Register** | Meaning |
| -------- | -------- |
| **The only...** | **...16 Registers DIRECTLY used by programs like C++** |
| `rax`,`rbx`, `rcx`, `rdx` | General purpose - Often for temp storage, return values, loop counters |
| `rsi`, `rdi` | - Used for function call arguments. Used for first 2 parameters of calling convention. Leftover from old x86, and repurposed rather than removed. `rsi` (source index to source data), `rdi` (destination index to dest memory) |
| `rbp` | The base pointer. (Frame pointer) - Holds the base address of the current stack frame. Used to track function calls? |
| `rsp` | The stack pointer. Holds the address FOREFRONT of the stack - the top (lower memory addresses as the stack grows downards). This will obviously be changing as the stack moves up and down. DONT GET CONFUSED ABOUT TOP vs BOTTOM. So when the function returns, `rsp` resets back down (higher mem address) to `rbp` |
| `r8`, ..., `r15` | 8 general purpose registers. There are no physical difference between these and the `rax` registers, this is simply the most efficient way to seperate things that compilers enjoy. |
| **The rest of...** | **...the registers that exist on the CPU but aren't directly used by C++** |
| `rip` | The 17th, an exception, Instruction pointer. It points to the address of the current instruction being executed, which is in the read-only text segment of memory |
| `eflags` | 32 bit size. Extended flags (also `rflags`, but `gdb` sometimes calls it the older name), stores the flags for arithemetic like overflow, or zero etc... |
| `-s` | These are all 'segment' registers, typically 32 bit or 16 bit. They are mostly obselete but used in legacy 32 bit architectures. BUT, there is a concept called Thread Local Storage which uses `fs` and `gs`. We keep these instead of simulating 32 bit with the larger 64 bit registers because older software has been designed to use these. Their removal breaks backwards compat. |
| `cs` | Code segment. Mostly obselete. Originally used to mark code memory |
| `ss` | Stack segment. Mostly obselete. Originally used to mark stack memory |
| `ds` | Data segment. Mostly obselete. Originally used to mark global/static memory (think data segment from modern mem arch) |
| `es` | Extra segment. Mostly obselete. Additional data segment |
| `fs` | Fifth segment. Now used for thread-local storage. I THINK its redundent for 64 bit, they use `fs_base` instead. Fifth and general were chosen for other roles because they were the least used for their previous role and therefore most available. | 
| `gs` | General segment. Now used for OS kernel storage. Fifth and general were chosen for other roles because they were the least used for their previous role and therefore most available.| 
| `fs_base` | Fifth segment base? Now used for storing the thread-local storage pointer |
| `fs_base` | General segment base? Now stores the per-CPU kernel data pointer |
| `k[0-7]` | Used to be used for security, but not used in any relevant modern setting i found |

#### Thread-local storage and Kernel Storage
The `fs_base`, and `gs_base` registers all have modern uses between these two. I may as well outline them a little here.
- Thread-local storage `fs_base`
    * In a multithreaded system, we'll want to have speedups while avoiding race conditions with some variables to get shared across threads.
    * To help with this, the OS gives each thread their own little section of memory that will mirror the structure of the other threads and each start at their own address `fs_base`.
| Thread   | FS_BASE (Start of TLS) | x (offset +0) | y (offset +32) |
| -------- | -------- | -------- | -------- |
| Thread 1 | 0x100000 | 0x100000 (x) | 0x100020 (y) |
| Thread 2 | 0x200000 |	0x200000 (x) | 0x200020 (y) |
| Thread 3 | 0x300000 |	0x300000 (x) | 0x300020 (y) |
| Thread 4 | 0x400000 |	0x400000 (x) | 0x400020 (y) |
- Kernel Storage `gs_base`
    * A similar concept to thread-local-storage, but shared between CPU cores for the kernel. An example of what they share would be the task queue.