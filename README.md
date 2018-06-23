# KrunkHook
KrunkHook is a hooking library, written in C++, for 32-bit and 64-bit Windows.

Functions can be hooked by calling **`Hook`**, and unhooked by calling **`Unhook`**.

**`TrampolineByFunc`** or **`TrampolineByHook`** can be used to call the original function while bypassing the hook. These statements return the same thing, but **`TrampolineByFunc`** takes the original function as an argument while **`TrampolineByHook`** takes the hook function as an argument.


## Example Usage:

```cpp
#include <Windows.h>
#include "krunk.h"

int WINAPI HookMessageBoxA(HWND hWnd, LPCTSTR lpText, LPCTSTR lpCaption, UINT uType)
{
	TrampolineByFunc(MessageBoxA)(hWnd, "Injected MessageBox, created using KrunkHook.", lpCaption, uType);
	return TrampolineByFunc(MessageBoxA)(hWnd, lpText, lpCaption, uType);
}

int main(int argc, char* argv[])
{
	Hook(MessageBoxA, HookMessageBoxA);
	MessageBoxA(0, "This is a default success message.", "KrunkHook Example", MB_OKCANCEL);

	Unhook(MessageBoxA);
	MessageBoxA(0, "This message should appear with no hook!", "KrunkHook Example", MB_OKCANCEL);
	return 0;
}
```

## Remarks
KrunkHook patches 1-byte of binary functions to intercept them.  An exception handler modifies the instruction pointer to the hook function when this patch is executed.

If a function begins with a branch instruction, or any instruction who's addressing is relative to the instruction pointer, the trampoline will fail.  This means trying to hook a function that is already  patched with some sort of jmp will not work.

Trampolines routines are built using **udis86** disassembler library to determine the instruction length of the patched instruction.  

**About Trampolines**
A trampoline is a small chunk of code which is allocated to allow calling the hooked function.
The patched instruction is executed by the trampoline and the trampoline jumps into the hooked function directly after the patch. This is how the hooked function is called without executing the hook.