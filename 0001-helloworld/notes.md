## 0001 - Hello World
### Compiling my first C++ script for a long while.
`g++ -o hw hw.cpp`
- `-o hw` is the name of the executable

### Best practice from now on:
`g++ -Wall -Wextra -Wpedantic -g -o hw hw.cpp`
- Extra warnings
- `-g` adds debugging symbols for use in GDB

### Also using Clang
`clang++ -o hw hw.cpp`

### Optimisation flags
`g++ -O2 -o hw hw.cpp`

### Header File and so libraries
#### Headers
`something.h`
- A header file contains declarations of classes, functions, and variables, that allow shared function between multiple `.cpp` files. But NOT instantiation of any of them
```cpp
// Example header file
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

int multiply(int a, int b);  // Function declaration (only tells the compiler it exists)

#endif
```
- The reason we use header files, and don't just directly include `.cpp` files, is that if many `.cpp` files all import from a shared `.cpp` file, then the intermediate stage would paste the entire content into each of them, significantly bloating the code
- I think this would actually also cause a multiple definition error for the final executable, which is just one file in the end of the day.
- AND, if i dont directly import say `file1.cpp`, then whenever I make a change to `file1.cpp` I only need to recompile and regenerate `file1.o` and NOT every other `.cpp` file that could have included it directly!!
#### Shared object files (.so)
Files that allow the use of compiled code across lots of programs without needing to recompile each time.
- Apparently these are linked at execution time? Unlike others which are linked at compile time


### The Entire Compilation Process:
#### 1. Preprocessing .cpp -> .i | Create intermediate
`.cpp -> .i`
- The preprocessor handles `#include`, `#define`, and marcos before the compiling
- Removes comments and expands `#define` macros
- Outputs pure C++, no macros or includes, just expanded code
- Can obvs view with anything cos its just C++
- Only manipulates text, no actual compilation
- The `.i` stands for intermediate
- What IS resolved:
    * `#include`, `#if`, macros
    * Any code from header files `.h`
- What IS NOT resolved:
    * Function *implementations* that exist in seperate source files (other `.cpp` files)
    * 'Any actual address resolution' (knowing exactly where in memory a function is located)
- Example of a file that linking hasn't happened yet
```cpp
#include <iostream>
#include "math_utils.h" // Declares multiply(), but doesn't define it

int main() {
    int result = multiply(3, 4); // Function is used here
    std::cout << "Result: " << result << std::endl;
    return 0;
}
```
- The preprocessed intermediate file would then have
```cpp
int multiply(int, int);  // From the header file
int main() {
    int result = multiply(3, 4);
    std::cout << "Result: " << result << std::endl;
    return 0;
}
```

- `g++ -E hw.cpp -o hw.i`
Example `hw.cpp`:
```cpp
#include <iostream>
#define PI 3.14159
int main() {
    std::cout << "Pi is: " << PI << std::endl;
    return 0;
}
```
Example `hw.i`:
```cpp
// <iostream> contents inserted here...
int main() {
    std::cout << "Pi is: " << 3.14159 << std::endl;
    return 0;
}
```

#### 2. Compilation .i -> .s | Create assembly
`.i -> .s`
- The compiler (g++) converts the preprocessed code into assembly
- Assembly is human readable machine instructions
- Can also cat the output, doesn't need anything fancy to dump
- the `.s` stands for 'source of assembly'
- Different ISAs create different assembly
- Can be minor variations inside ISAs depending on micro architecture
- The compiler checks function signatures from header files here to make sure there aren't errors relating to this
- `g++ -S hw.i -o hw.s`
Example `hw.s`
```
.globl main
main:
    pushq   %rbp
    movq    %rsp, %rbp
    movl    $0, %eax
    popq    %rbp
    ret
```

#### 3. Assembling .s -> .o | Create machine code
`.s -> .o`
- the `.o` stands for object file
- The assembler then converts the assembly into raw machine code
- I think this is where the micro architecture starts to come in?
- With the correct instruction set, machine code can be converted back into assembly almost always
- `g++ -c hw.s -o hw.o`
- `objdump -d hw.o # Disassemble to see machine code`
- `hexdump -C hw.o # to view raw binary`

#### 4. Linker .o -> executable | Creates the executable
- `.o -> executable`
- Resolves references to external functions like `printf`
- Adds necessary libraries like C++ stndard
- `g++ hw.o -o hw`
- `file hw # shows the type of executable`
    * `hw: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2`
    * e.g. dynmically linked, on x86-64 architecture, etc...
- `ldd hw # shows linked libraries`
- Ok, so the linker here needs to take all the created `.o` files one at a time. This is because we WANT to have everything compiled seperately (it makes editing code easier and not having to compile the same function multiple times)

#### 5. Extra libraries | .so files (shared object) and dll (dynamic libraries)
- Where an `.o` object file is compiled, but as of yet unlinkedi. They have declarations but undefined references (including std::out)
- `.so` files (or `.dll` in windows) are standalone files that are fully linked and compiled, but yet to be baked into any executable.
- A dynamic linker loads `.so` files into memory at runtime
- I can check which shared library my executable needs using `ldd my_executable`
- Here is a scenario about why you need `.so` files, and why they should be precompiled and linked
    1. Shared math function 
    ```cpp
    // math.cpp
    #include <iostream>

    extern "C" int add(int a, int b) {  // 'extern "C"' prevents C++ name mangling
        return a + b;
    }
    ```
    2. `g++ -shared -fPIC math.cpp -o libmath.so`
        * `-shared` makes a shared object
        * `-fPIC` 'Position independent code' that lets multiple programs use it safely
            - Position independent code is machine code that can run at any memory address, rather than fixed locations
            - Since `.so` files can be anywhre in memory becuase it happens at run time, there needs to be allowances for this
            - We DONT make all code position independent because there are overheads and costs associated with it, most code doesn't need it
            - Relying on them would increase security risks
    3. Program using the `.so`
    ```cpp
    // main.cpp
    #include <iostream>

    extern "C" int add(int, int); // Declare the function from the shared library

    int main() {
        int result = add(5, 7);
        std::cout << "Result: " << result << std::endl;
        return 0;
    }
    ```
    4. Compile `main.cpp` with `.so` without using `math.cpp`
        * `g++ main.cpp -L. -lmath -o my_program`
        * `-L.` to look for `.so` files in the current directory
    
    5. If i ran this, I'd get an error finding libraries
        * I'd need to update `export LD_LIBRARY_PATH=.` to help it
- When linking an external library, the linker automatically adds `lib` and `.so` where searching for them because of naming converntion
    * Here, `libmath.so` is the name of the library
    * But the compiler is written to let you just type `-lmath` to save space and time     
    

### Questions I had:
1. do i add extra warnings and debug symbols with clang.
    * Yes
2. What EXACTLY are debugging symbols?
    * Extra metadata in the compiled binary that map machine instructions back to C++ source code
3. I remove `-g` at time of deployment right? For what reason?
    * Smaller executables, and security, allowing attackers to reverse engineer the code
4. What does the compiled process do?
    * 1) Preprocessing, getting the #include and #defines
    * 2) g++ or clang converts the C++ into assembly code `.s`
    * 3) The assembler turns assembly code into binary machine code `.o`
    * 4) The linker connects functions, libraries, and system calls into the executable `hw`
5. To what extent does the debugger need the original C++ source if i have allowed debug symbols during compilation?
    * You don't need them, but some clever debuggers will look for it and display the line of code itself if its there. If the originalal `.cpp` file is missing, then it simply gives you the symbols it has in the binary.
6. Remind me the exact intuition behind the name 'binary' file
    * Generally means not human readable. It is straight up raw machine code. 1s and 0s
7. What kinds of optimisations do optimisation flags do?
    * Removing unused code
    * inlining small functions
    * 'loop unrolling'
    * Smarter usage of registers
8. Can optimisations introduce erros? Should i be careful?
    * Optimisations should NOT add errors to correct code
    * BUT optimisation flags can mess with debugging because it will remove and rearrange code
    * Old CPUS might not support loop vectorisation
9. While i'm where, what was shebang again `#!/bin/bash`
    * It tells the OS to execute the script with the specified shell. Could be fsh or zsh too.
    * `.cpp` files don't need shebangs because its already machine code, the shell instead throws it straight at the GPU, without needing an interpreter like is typical in bash or python
10. Why does assembly have the suffix `.s`?
    * It stands for source, i'm actually just gonna make a whole table for this in the above bit
11. To what extent is machine code dependent on architectures:
    * Compiled binaries ARE CPU dependent
    * Instruction set architecture (ISA) vs Microarchitecture
        - ISA is the high level contract that all CPUs share, e.g. ARMv8, x86-64
        - Microarchitecture is the specific implementation details inside families. E.g. Intel Skylake and AMD Zen 3 both use x86-64
        - Generally, shared ISAs can run the same unoptimised machine code
        - Optimised binaries may not work between ISAs (`-march=native`)
    * This means that g++ and clang are aware of my exact CPU model and they generate optimised machine code for it
    * Here is my instruction set `gcc -march=native -Q --help=target`
        - This gave me a huge list of things that are enabled or disabled
    * Compiling for maximum portability: `g++ -march=x86-64 -mtune=generic -O2 hw.cpp -o hw`
        - `-march=x86-64` ensures compatability across all x86-64 CPUs
        - `-mtune=generic` optimised for general performance across different CPUs
        - `-O2` flag uses safe optimisations without CPU specific instructions?
    * `-march` stands for machine architecture. Does the ISA (less specific, unless set to native)
    * `-mtune` stands for tuning. Does the more specific microarchitecure
12. What exactly is undefined behaviour, and why have it?
    * `int x = 42; std::cout << x; // x is defined`
    * `int x; std::cout << x; // x is NOT defined`
    * We have UB cos then the compiler doesn't need to do safety checks for each option
    * When unoptimised `-O0`, `x` will still be hanging around, and it'll probably contain random leftover memory from the stack.
    * Optimisers e.g.`-O2` will realise its not defined and cut it out, leading to weird things happening
13. 