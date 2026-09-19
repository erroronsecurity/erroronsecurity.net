# Simple Bind Shell

Today, we're reviewing a simple bind shell, specifically the one found at [https://github.com/erroronsecurity/misc/bind_shell.c](https://github.com/erroronsecurity/misc/bind_shell.c).


First, we need a couple of libraries for the functions we'll be using:
```
#include <stdio.h>
#include <string.h>

#include <sys/socket.h>
#include <netinet/in.h>
#include <apra/inet.h>
...
```

stdio.h and string.h are used for printing, popen, memset, and sizeof.
socket.h, in.h, and inet.h are our network libraries (on linux).

Create the main function:
```
int main(void) {
...
```

Create the listening socket:
```
