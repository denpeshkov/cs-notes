---
tags:
  - Go
  - OS-Architecture
  - TODO
  - Networking
---

# Overview

Netpoller sits in its own thread, receiving events from goroutines wishing to do [network](Network.md) I/O. It uses `epoll` [system call](System%20Calls.md) to do polling of network sockets (file descriptors)

Whenever you `open` or `accept` a connection in Go, the file descriptor that backs it is set to non-blocking mode. This means that if you try to do I/O on it and the file descriptor isn't ready, it will return an error code saying so

Whenever a goroutine tries to read or write to a connection, the networking code will do the operation until it receives such an error, then call into the netpoller, telling it to notify the goroutine when it is ready to perform I/O again. The goroutine is then scheduled out of the [`M`](Go%20Goroutines%20and%20Scheduler%20Internals.md) it's running on and another goroutine is run in its place

When the netpoller receives notification from the OS that it can perform I/O on a file descriptor, it will look through its internal data structure, see if there are any goroutines that are blocked on that file and notify them if there are any. The goroutine can then retry the I/O operation that caused it to block and succeed in doing so

# References

- [The Go netpoller - Morsing's blog](https://morsmachine.dk/netpoller)
