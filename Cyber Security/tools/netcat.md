### What does it do
`netcat` or `nc` is a utility that allows us to interact with TCP/UDP ports. Its primary usage is for connecting to shells. It can also connect to any listening port and interact with the service running on that port. For example:

```bash
user@htb[/htb]$ nc 10.10.10.10 22 
SSH-2.0-OpenSSH_8.4p1 Debian-3
```
> Connecting to port 22 with `netcat` lets us discover that `SSH` is running on it!

`netcat` can also be used to transfer files between machines, which we'll see later (unless I come back here)
`socat` is a similar utility with some extra features, such as port forwarding and connecting to serial devices. It can also be used to upgrade a shell to a TTY.
