## Debugging and Memory Architecture
I'm going to run `gdb` and get used to using it

Note, the `gdbinit` here is from the gdb-dashboard package on the AUR. I don't own it. ee the file for usage rights etc. Its just here to be helpful and for me to document my progress

### Stripping debug symbols
- I can actually strip a binary thats already created with debug symbols
- `strip hw`
- Strip removes more than debug symbols, but broadly it is mostly just used to remove debug symbols
- You CAN strip an ELF compiled without debug symbols and it will remove stuff (i proved it here), so can be useful to squeeze a bit of extra size out of an ELF
- Theres more to strip, but this is all i need to know for now

### Running gdb
- So i tried `cgdb` and messing with my `.gdbinit`, for now i'm using `gdb-dashboard` from GitHub
1. `gdb ./hw` 
2. `run` - Start the run
3. `break main` puts a breakpoint at main
4. `step` `next` `continue` work like they do in Python
5. `info registers` i remember this from security project in undergrad
6. `disassemble main`
7. Can use conditional breakpoints
    * `break main if x == 42`
8. Inspecting call stack
    * `backtrace`
9. Setting variables through execution
    * `set variable x = 100`

### Memory Architecture
I know most of this from linux stuff and comp systems in UG
- Register < L1 < L2 < L3 < RAM < SSD etc...
    * Specifically, reigsters inside the CPU
    * Most L1 caches near
- Stack vs Heap:
    * [Read this](https://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap). I seriously couldn't explain it better
- The Stack:
    * Starts at high memory and grows down `0xfffff` -> `0x00000`
    * The fast and 'organised one'. Think a stack of books
    * Porition of LIFO memory allocated at process startup and maintained by that process
    * Local variables and function calls
    * Generally put in registers, making it super fast
    * Few MB per thread roughly
    * Doesn't need manual deallocation
    * Every thread is given a stack by OS
    * Since it grows and shrinks predictably, one never needs to search for free space?
- The Heap:
    * Slower, think of a disorganised heap of objects
    * (new malloc) slower because they involve OS-level mem management. Stack allocations are just pointed adjustments
    * Global variables
    * Dynamic memory allocation
    * No fixed size limits
    * Heap is generally given to application in general
- Multithread
    * Each thread gets its own stack
    * But all threads share heap (why you need to avoid race conditions when allocating memory)

#### Stack Frames
This is well worth understanding now exactly how the stack and heap are doing things.
- Think of function calls in a program as having a frame in the stack
- The compiler figures out the memory each variable will need in a given frame and fits them all in. There are offsets for each variable. 
- Each function ONLY needs to know the base pointer for its current function (`rbp`) and the base pointer `rbp` for the previous function.
    * Since each thread has its own stack, think about it, a function needs to know only details about itself, and details about the last function that called it
    * Its like a depth first search
- The reason the stack is last in first out is because its like a depth-first search. Main is the very start, and each recursive subroutine is 1 deeper, which we occasionally pop out of.
```
High memory  ───────────────>  Stack starts here  
            | main() frame  |  ← Highest address in stack  
            | funcA() frame |  
            | funcB() frame |  
Low memory  ───────────────>  Stack grows downward 
```
- **The stack goes from high to low memory because:**
    * Lots of reasons historically apparently.
    * For a given allocation of memory, put all your code at the end where its fixed and let the heap grow up into it

- **What Happens if the Heap runs out of memory?:**
    * We have the concept of virtual memory handled by the OS to thank for that. All that happens there is the physical memory of the stack may be moved, but the virtual memory address is kept fixed.
```
High Memory (end of address space)
-------------------
|    Stack        |
|    (grows down) |
------------------- 
|                 |
|                 |  
------------------- 
|    Heap         |
|    (grows up)   |
------------------- 
Low Memory (start of address space)
```

- **What is stored in the stack?**:
    * Small local vars, `int`, `double`, small structs
    * Return addresses and saved registers (so we can return)
    * Pointers to heap allocated objects when they're too big for the stack

- **What happens when huge objects are created?**:
    * If we allow it to be stored on the stack, it would be too large and cause a stack overflow
        + `int x = 42;   // Stored directly in the stack`
    * So we need to explicitly allocate it to the heap
        + `int* p = new int[1000000];  // Stored in the heap, p holds the address`
    * If you use `new`, you must manually delete it later
    * Here's a memory bug
```cpp
int* p = new int[10];  // Heap allocation
delete[] p;            // Memory freed
p[0] = 42;             // ❌ Use-after-free bug! Undefined behavior!
```





### What Questions Did I Have?
- What is an ELF exactly?
    * An Executable and Linkable Format is a standard format for binary files with shared libraries objects `.so` and objects `.o` for linux.
    * Basically all C++ executables on linux are ELF
- At what point exactly in compilation process are debug symbols added?
    * I guessed this one correctly, but its at the assembly stage, where `.s` -> `.o`
    * `g++ -c -g hw.s -o hw.o`
        + `-g` is the right flag
- Can I run gdb effectively on an ELF binary without debug symbols?
    * Yeah you can still
        + Disassemble
        + Set breakpoints
        + Step through execution
        + Analyse stack frames
    * But...
        - No function or variable names
        - No source code mapping (which line in the main C++ machine instructions correspond to)
- Do Devs use GDB or something better?
    * On Linux, a lot use `gdb` still
    * But consider `cgdb` for a text UI
    * `gdb-dashboard` for a UI
