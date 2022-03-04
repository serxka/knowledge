# SysV ABI
The System V Application Binary Interface is a calling convention that is used on the majority of x86\_64 software (bar Windows of course). It is also adaptable to other architectures, however it's most applicable to x86\_64 as that is the most common architecture it applies to.

## x86\_64
Before the `call` instruction, the stack **must** be alinged to a 16-byte boundary. The direction flag (`DF`) in the RFLAGS register must also be cleared. There is also a 128-byte "red zone" below the stack frame which is typically used by leaf functions (ones that do not call others) for stack allocation without moving the stack pointer, this saves an instruction.

### Function calls
|    Purpose   | Register |
|:------------:|:--------:|
|  Parameters  |    rdi   |
|              |    rsi   |
|              |    rdc   |
|              |    rcx   |
|              |    r8    |
|              |    r9    |
| Caller-saved |    r10   |
|              |    r11   |
|    Return    |    rax   |
|              |    Rdx   |
| Callee-saved |    rbx   |
|              |    rsp   |
|              |    rbp   |
|              |    r12   |
|              |    r13   |
|              |    r14   |
|              |    r15   |

### Linux System Calls
It is pretty similar with system calls however the roles of `rcx` and `r10` are swapped. Calls are made using the `syscall` instruction rather than `call`. The reason `rcx` is swapped as it gets destroyed by kernel as part of the `syscall` instruction semantics.
|    Purpose   | Register |
|:------------:|:--------:|
|  Parameters  |    rdi   |
|              |    rsi   |
|              |    rdc   |
|              |  **r10** |
|              |    r8    |
|              |    r9    |
| Caller-saved |  **rcx** |
|              |    r11   |
|    Return    |    rax   |
|              |    Rdx   |
| Callee-saved |    rbx   |
|              |    rsp   |
|              |    rbp   |
|              |    r12   |
|              |    r13   |
|              |    r14   |
|              |    r15   |


[SysV ABI AMD64 Docs](https://www.uclibc.org/docs/psABI-x86_64.pdf)
