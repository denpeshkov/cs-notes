---
tags:
  - Networking
  - TODO
---

# Connection-Oriented Flow

When a UDP socket is created, its local and remote addresses are unspecified. Datagrams can be sent immediately using `sendto()` or `sendmsg()` [system calls](System%20Calls.md) with a valid destination address as an argument

When `connect()` is called on the socket, the default destination address is set and datagrams can now be sent using `send()` or `write()` without specifying a destination address. It is still possible to send to other destinations by passing an address to `sendto()` or `sendmsg()`

The `connect()` [system call](System%20Calls.md) connects the socket to the specified address. If the socket is of type `SOCK_DGRAM`, then the specified address is the address to which datagrams are sent by default, and the only address from which datagrams are received

# References

- [udp(7) - Linux manual page](https://man7.org/linux/man-pages/man7/udp.7.html)
- [connect(2) - Linux manual page](https://man7.org/linux/man-pages/man2/connect.2.html)
- [Computer Networking A Top Down Approach. James F. Kurose, Keith W. Ross](References.md#Computer%20Networking%20A%20Top%20Down%20Approach.%20James%20F.%20Kurose,%20Keith%20W.%20Ross)
- [Everything you ever wanted to know about UDP sockets but were afraid to ask, part 1](https://blog.cloudflare.com/everything-you-ever-wanted-to-know-about-udp-sockets-but-were-afraid-to-ask-part-1)
