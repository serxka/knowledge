# GNU Builtins
GCC and Clang provide a bunch of helpful builtin functions such as rotating bit-shifts or byte-swaps that can use intrinsics or special compiler heuristics to help speed them up. These functions are prefixed with the standard two underscores of compiler keywords and the builtin word: `__builtin_<func>`. Very commonly these will be math implementations such as `sin` or `sqrt` but there are also bit operations like `parity` or `bswap` and `rotl`. You can view a list of these kinds of builtin functions in the [GCC docs](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html). These functions will often be suffixed by how many bits they take to operate, such as `__builtin_rotl32` which will rotate the bits of a 32-bit integer (4-bytes) to left however many places as passed to the function.
	
Aside from these builtin functions there are also ones which leverage knowledge only know at compile time to help speed things up. A common example of this is `memcpy`, in which `__builtin_memcpy` will perform special function such as using special registers or unrolling a loop for certain small sizes. A mock example of what this might look like in the compiler:
```c
void generate_builtin_memcpy(expr dest, expr src, expr size) {
	if (is_constant(size) && eval(size) < MEMCPY_INLINE_THRES) {
		// perform compiler magic to get a fast memcpy for small size
	} else {
		// fallback to standard memcpy
	}
}
```
This builtin `memcpy` also applies for similar memory functions such as `memmove` and `memset`. An important thing to note on how the compiler will fallback to standard memcpy is that it can fallback to just calling the symbol. This means if you create your own `memcpy` the builtin memcpy can fall back to that one. This is of note if you have code written like below (note: this code is incorrect and can stack overflow)
```c
void *memcpy(void *restrict dest, const void *restrict src, size_t n) {
	// __builtin_memcpy may possibly call `memcpy` (this function!)
	return __builtin_memcpy(dest, src, n);
}
```
