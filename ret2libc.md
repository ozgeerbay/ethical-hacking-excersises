# ret2libc (Return-to-libc) – NX Bypass Concept

## Goal
Spawn a shell **without executing shellcode on the stack** by redirecting control flow to a libc function (typically `system`) and passing `"/bin/sh"` as its argument.

This is commonly used to bypass **NX (non-executable stack)** because no code is executed from the stack.

---

## Vulnerable Program (example)
```c
int main(int argc, char *argv[]) {
    char buffer[5];
    strcpy(buffer, argv[1]);
    return 0;
}
```
The overflow allows overwriting the saved return address.

## Why ret2libc Works (short)

On x86, a function call using a stack-based calling convention expects:
```bash
[RET to system] [RET after system] [arg1] [arg2] ...
```

For system("/bin/sh") we need:
-Return address overwritten with the address of system()
-A fake return address (anything valid-ish) after system returns
-A pointer to a "/bin/sh" string as the first argument

## Step 1 - Find the offset to saved return address

Method: send increasing patterns and observe crash location.

Evidence you typically include:
- program crashes at length N
- saved return address gets overwritten at offset OFFSET = ______

(Example approach)

- Try repeated pattern blocks and binary-search the crash point.
- Confirm in a debugger by inspecting the stack at crash time.

## Step 2 - Resolve system() address in libc

In your notes you had:

- breakpoint at main
- print system

Debugger evidence format:
```bash
(gdb) break main
(gdb) run
(gdb) print system
$1 = 0x________ <system>
```
Record the system address.

## Step 3 - Obtain a "/bin/sh" pointer

Two common approaches:
A) Use libc’s own "/bin/sh" string:
In gdb you can search for it (your note: search-pattern "/bin/sh").

B) Store "/bin/sh" in an environment variable:
This is a classic trick in older labs because it’s easy to reference.

## Step 4 - Build the ret2libc stack layout

Final stack layout (x86 conceptual):
```bash
padding (OFFSET bytes)
SYSTEM_ADDR
FAKE_RET
BINSH_ADDR
```

## Step 5 - Run and verify

```bash
$ ./vuln <crafted_input>
$ whoami
root
```

