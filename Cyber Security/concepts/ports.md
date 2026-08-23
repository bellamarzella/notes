## What is a Port
A port is like a window or door on a building, if the building is a remote system. A 24/7 McDonalds has to leave their front door open at all times to let customers in, but keeps the window to the back office locked at all times so that no one can climb in. 

Ports are virtual points where network connections begin and end. They are software-based and managed by the OS. Each port is associated with a specific process or service and allow machines to differentiate between different traffic types (for example, `SSH` traffic flows to a different port than web requests even though they both use the same connection.)

Each port is assigned a number, and many are standardised. For example, `HTTP` messages go through port `80`, unless configured otherwise. Port 0 is reserved for TCP/IP networking. If anything attempts to bind to port 0, it is instead bound to the next available port above 1,024. Thus, 0 is treated as a *wildcard* port.
## Two types of Ports
### Transmission Control Protocol (TCP)
`TCP` is connection-oriented, meaning a connection between client and server must be established before data can be sent. The server must remain in a listening state to wait for connections from clients.
### User Datagram Protocol (UDP)
`UDP` utilises a connectionless model. There is no agreed upon connection, which introduces unreliability as there is no guarantee that data will actually be delivered. `UDP` is useful when error checking/correction isn't needed or can be handled by the application. 
It is suitable for time-sensitive tasks, as it is faster to drop packets than wait for delayed packets.

There are 65,535 of each type of port. These are some of the most well-known.

| Port(s)         | Protocol              |
| --------------- | --------------------- |
| `20`/`21` (TCP) | `FTP`                 |
| `22` (TCP)      | `SSH`                 |
| `23` (TCP)      | `Telnet`              |
| `25` (TCP)      | `SMTP`                |
| `80` (TCP)      | `HTTP`                |
| `161` (TCP/UDP) | `SNMP`                |
| `389` (TCP/UDP) | `LDAP`                |
| `443` (TCP)     | `SSL`/`TLS` (`HTTPS`) |
| `445` (TCP)     | `SMB`                 |
| `3389` (TCP)    | `RDP`                 |