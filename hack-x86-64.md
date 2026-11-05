# Bridge note: Hack assembly to x86-64
 
Hack is a tiny CPU I already know. x86-64 runs modern operating systems (Windows, Linux, macOS) and heavy applications.
 
## 1. Registers
 
| Hack | x86-64 | Note |
|---|---|---|
| `D` (data) | `rax`, `rbx`, `rcx`, `rdx`, `rsi`, `rdi`, `r8`–`r15` | x86 has many general registers, not one |
| `A` (address/value) | any register used as a pointer | `[rax]` means "memory at rax" |
| `M` (= RAM[A]) | `[address]` | x86 writes the memory access explicitly |
| `SP` (RAM[0], by convention) | `rsp` | a real register, built into the CPU |
| 16-bit words | 64-bit registers, byte-addressed memory | x86 is little-endian |
 
## 2. Load, add, store
 
```
Hack                x86-64 (Intel syntax, what Ghidra shows)
@5                  mov rax, 5
D=A
@x                  mov rax, [x]
D=M
@y                  add rax, [y]
D=D+M
@z                  mov [z], rax
M=D
```
 
## 3. Compare and jump
 
Hack has no compare instruction: subtract, then jump on the result.
x86 splits it in two: `cmp` sets flags, then a `jcc` reads them.
 
```
Hack                x86-64
@5                  cmp rax, 5
D=D-A
@DONE               je DONE
D;JEQ
@LOOP               jmp LOOP
0;JMP
```
 
| Hack | x86-64 (signed) |
|---|---|
| `JEQ` / `JNE` | `je` / `jne` |
| `JGT` / `JGE` | `jg` / `jge` |
| `JLT` / `JLE` | `jl` / `jle` |
| `JMP` | `jmp` |
 
New in x86: `ja`/`jb` are the *unsigned* versions. Ghidra shows these a lot.
 
## 4. The stack
 
In Nand2Tetris the stack was **software**: my VM translator emitted the code
that moves `SP`. In x86 it's **hardware**.
 
| | Hack VM | x86-64 |
|---|---|---|
| push | `@SP A=M M=D @SP M=M+1` | `push rax` |
| pop | `@SP AM=M-1 D=M` | `pop rax` |
| direction | grows **up** (SP++) | grows **down** (rsp -= 8) |
| call | I pushed the return address myself | `call f` pushes it for me |
| return | I jumped to the saved address myself | `ret` pops it and jumps |
 
Function frames use `rbp`/`rsp`, just like the VM's `LCL`/`ARG` pointers.
 
## 5. Calling convention (on Ghidra)
 
On Linux x86-64, arguments go in registers, not on the stack:
 
`rdi, rsi, rdx, rcx, r8, r9` for arguments 1 to 6.  The return value is in `rax`.
 
So `mov edi, 5 ; call f` means `f(5)`.
 
## Cheat sheet to keep open in Ghidra
 
- `mov a, b` copies b into a
- `lea a, [b+c]` computes an address, with no memory read
- `xor eax, eax` sets eax to 0
- `test eax, eax` + `je` means "if eax == 0"
- `call`/`ret` are the function boundaries
