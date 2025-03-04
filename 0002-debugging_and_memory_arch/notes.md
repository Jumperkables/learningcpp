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
