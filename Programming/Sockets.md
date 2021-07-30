# Sockets Programming
Sockets a "magic data portal" where you can stuff some data in and have it come out at another place. Instead put nicely, you can _write_ to a file descriptor and then _read_ from that data from another file descriptor. However this data is sent between two processes or more common between two computers on a network. In another form, it is an IPC channel between processes (irrespective of the machine they run on), transported either through memory (Unix Sockets) or over wire using network protocols such as TCP or UDP.

## Socket Creation
A socket file descriptor is first create through the system call `socket`, this takes parameters about the kind of socket required. It will first an enum defining the communication domain we want to operate on, most common for this is `AF_UNIX` for raw Unix sockets or `AF_INET` for an IPv4 Protocol. After this takes a type specifying the style of communication which should take place (think OSI Layer 4), the three most common options are `SOCK_STREAM` for TCP, `SOCK_DGRAM` for UDP and `SOCK_RAW` for just raw data like in Unix Sockets. Last is protocol which isn't super relevant, we will set it to `0`. Refer to the `man` page for more details.

```c
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```

#### Server Setup
The general setup to create a socket in which clients can connect to is fairly simple. We create our `socket`, then we `bind` it to an address, we then set how many connects we want to `listen` for, we can then poll in a loop and `accept` and incoming connections to get an file descriptor. With these file descriptors we can then read and write to them. A common method taken in servers is to `accept` a certain amount of connections and then poll on all them for available data using the Linux system call `epoll`, or `kqueue` on BSD and MacOS.

#### Client Setup
Connecting to a socket is very simple. We first create our `socket` and then `connect` to and address, we then get a file descriptor we can read and write to.

### Example
Here is an example of what an echo server might look like
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>

#define error(msg) do { perror(msg); \
                        exit(EXIT_FAILURE);} while (0);
int handle_conn(int fd) {
	unsigned char buf[256];
	memset(buf, 0, 256);
	int n = read(fd, buf, 256);
	if (n < 0)
		error("read");
	n = write(fd, buf, n);
	if (n < 0)
		error("write");
	close(fd);
	return strncmp(buf, "exit", 4) == 0;
}

int main(void) {
	// Get our socket file descriptor
	int sock_fd = socket(AF_INET, SOCK_STREAM, 0);
	// Check for errors
	if (sock_fd < 0)
		error("socket");
	// Create a structure representing how we want to bind our socket
	struct sockaddr_in addr = {
		AF_INET, // Should always be set to AF_INET
		htons(1234), // Our port, we convert it to network byte order
		INADDR_ANY, // The address we wish to bind, our hosts address
		{0} // Must be zero
	};
	// Bind our address to the socket
	if (bind(sock_fd, &addr, sizeof(addr)))
		error("bind");
	// Listen on our socket, we set the fd and amount of connects we will
	// backlog before returning an error, this cannot fail if fd is valid
	listen(sock_fd, 5);
	int addr_len = sizeof(addr);
	// Continuously accept incoming connections
	while (1) {
		// Get a file descriptor for this new connection
		int conn_fd = accept(sock_fd, &addr, &addr_len);
		// Check for errors
		if (conn_fd < 0)
			error("accept");
		// Handle it and echo back any data
		if (handle_conn(conn_fd))
			break;
	}
	close(sock_fd);
}
```
Along with what our client might look like:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>

#define error(msg) do { perror(msg); \
                        exit(EXIT_FAILURE);} while (0);
int main(void) {
	int sock_fd = socket(AF_INET, SOCK_STREAM, 0);
	if (sock_fd < 0)
		error("socket");
	struct sockaddr_in addr = {
		AF_INET,
		htons(1234),
		INADDR_ANY,
		{0}
	};
	if (connect(sock_fd, &addr, sizeof(addr)) < 0)
		error("connect");
	while (1) {
		char s[256];
		fgets(s, 256, stdin);
		int len = strlen(s);
		printf("%d\n", len);
		if (len == 0)
			continue;
		int n = write(sock_fd, s, len);
		if (n <= 0)
			error("write");
		n = read(sock_fd, s, len);
		write(STDOUT_FILENO, s, len);
		if (strncmp(s, "exit", 4) == 0)
			break;
	}
	close(sock_fd);
}
```