# Assembler Dialects
There are two different main assembler dialects	`AT&T` and `INTEL`. AT&T is the painful older one which is used almost exclusivly by GCC and Clang. INTEL is the cleaner one which is more legible and is used by pretty much everything else.

The general form of AT&T instructions are,
```
mnemonic source, destination
```
whereas for the INTEL syntax they are like this:
```
mnemonic destination, source
```

All examples have a relative comparsion in the other dialect subsections

## AT&T
#### Registers
The names of all registers must be prefixed with a `%`, for example `%eax`, `%rip`, `%cr3`
```asm
mov %rax, %rbx  # move contents of rax to rbx
```
#### Literals
Literals must be prefixed with a `$` , for example `$0x10`, `$10`, `$-123798`
```asm
mov %rcx, $0x80 # moves the value 127 into rcx
mov %rax, $512  # moves the value 512 into rax
add $10, %rdi   # adds 10 to the content of rdi
```
#### Memory Addressing
Memory addressing AT&T syntax is a fucking bitch, most painful shit ever, but it is done in the following way: `segement:offset(base, index, scale)`, parts of this can be omitted depending on if the are needed. Scale is the exponent in `2^x`. For example `%fs:0x8(%esi,%ecx,2)`
```asm
100             # memory at 100 byte offset from start of %ds segement register
%es:100         # memory at 100 byte offset from start of %es segement register
(%eax)          # memory at address in %eax
(%eax,%ebx)     # memory at address %eax + %ebx
(%ecx,%ebx,2)   # memory at address %eax + %ebx * 2
(,%ebx,2)       # memory at %ds segement offset of %ebx * 2
-10(%eax)       # memory at address in %eax - 10
%ds:-10(%ebp)   # memory relative to %ds at address in %ebp - 10
```
#### Operand sizes
You will want to specify the size of what is being moved into memory, for example `mov $10, (%rdi)` will move the value into memory at `%rdi`, but how large? This is done with suffixs on the mnemonic so it knows, for example `movd $10, (%rdi)` will move a 32bit value of 10 into memory.
```asm
movb $10, (%rdi) # b suffix denotes a byte (8bits)
movw $10, (%rdi) # w suffix denotes a word (16bits)
movd $10, (%rdi) # d suffix denotes a double word (32bits)
movq $10, (%rdi) # q suffix denotes a quad word (64bits)
```

## INTEL
#### Registers
The names of registers can be typed out as regular, for example `eax`, `rip`, `cr3`
```asm
mov rbx, rax    # moves the content of rax to rbx
```
#### Literals
Literals can be written without any notation of prefixing, for example `0x10`, `10`, `-123798`
```asm
mov 0x80, rcx   # moves the value 127 into rcx
mov 512, rax    # moves the value 512 into rax
add rdi, 10     # adds 10 to the content of rdi
```
#### Memory Addressing
Memory addressing in INTEL syntax is much fucking nicer, it is done in the following way: `[segement:base+index*scale(+/-)offset]`, parts of this can be omitted depending on if the are needed. Scale is the exponent in `2^x` For example `[fs:esi+ecx*2+0x8]`
```asm
[100]           # memory at 100 byte offset from start of ds segement register
[es:100]        # memory at 100 byte offset from start of es segement register
[eax]           # memory at address in eax
[eax+ebx]       # memory at address eax + ebx
[eax+ebx*2]     # memory at address eax + ebx * 2
[ebx*2]         # memory at ds segement offset of ebx * 2
[eax-10]        # memory at address in eax - 10
[ds:ebp-10]     # memory relative to ds at address in ebp - 10
```

Examples inspired by [this article](http://www.sig9.com/articles/att-syntax)
