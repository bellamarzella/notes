# ssh
## What does it do
Basically a legitimate bind-shell. Allows us to connect to a server and run commands as if it were our own machine.
## Usage
```bash
ssh [username]@[hostname or IP address]
```
## Flags
- `-p [port number]`: If `SSH` isn't the default 22, we can specify which port we want to connect to.
- `-i [path to key file]`: If the server verifies the user with a private RSA key instead of a password, we can specify the file.
## netcat
### What does it do
`netcat` or `nc` is a utility that allows us to interact with TCP/UDP ports. Its primary usage is for connecting to shells. It can also connect to any listening port and interact with the service running on that port. For example:
```bash
user@htb[/htb]$ nc 10.10.10.10 22 
SSH-2.0-OpenSSH_8.4p1 Debian-3
```
Connecting to port 22 with `netcat` lets us discover that `SSH` is running on it!

`netcat` can also be used to transfer files between machines, which we'll see later (unless I come back here)
`socat` is a similar utility with some extra features, such as port forwarding and connecting to serial devices. It can also be used to upgrade a shell to a TTY.
## nmap
### What does it do
Nmap is the primary reconnaissance tool. It tells us:
1. **Which ports are open.**
2. **What services are running on those ports**: `SSH`. `HTTP` etc.
3. **The version of services:** Finds the exact software build (`Apache httpd 2.4.41`), which we can then use to find exploits
### Usage
#### Step 0: Syntax
```bash
nmap [FLAGS] [TARGET_IP]
```
#### Step 1:  Fast map
```bash
nmap -p- --min-rate 5000 -v [TARGET_IP]
```
Scans all possible ports at 5000 packets per second minimum and show me results as they happen
#### Step 2:  Deep dive
```bash
nmap -sV -sC -p [PORTS] [TARGET_IP]
```
Find the version and run default nmap scripts on these specific ports

### Flags
#### Efficiency
- `-p-`: Scan all 65,535 possible ports. By default nmap only scans the 1000 most common
- `--min-rate n`: Forces nmap to send at least $n$ packets per second. Runs faster, however can result in rate limiting and/or missed ports if $n$ is too high. 
	- *Ports would be missed if the server is unable to handle all of the packets and drops them, causing it to look like the port is closed.*
#### Information
- `-sV` - **Service Version** detection: Finds exact software name and version string.
- `-sC`- **Default Scripts**: Finds more information about open ports by performing automated tasks; like reading webpage titles or checking for anonymous logins. 
#### Scope and Output
- `-p [PORT]`: Restricts scan to specified ports.
- `-v` - **Verbose Mode**: Prints ports as they are discovered in real-time, as opposed to waiting for the scan to finish.

