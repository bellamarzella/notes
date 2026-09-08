In order to begin exploring a machine, we need to identify the operating system and any services running on it. We're especially interested in services that have been misconfigured or that have a known vulnerability, so that we can coerce the service into performing some unintended action in our favour.

Computers are assigned an IP address, which allows them to be uniquely identified and accessible on a network. Services may be assigned a [[ports|port]] to make the service accessible. 

To access a service remotely, we connect using the correct IP and port. Manually examining all 65,535 ports would be impossible, so we use [[nmap]].

## Attacking Network Services
### Banner Grabbing
Banner grabbing is a useful technique to identify a service quickly. Often, a service will identify itself by displaying a banner when a connection is initiated. nmap will attempt to grab the banners with the flag `--script-banner`, but this is included in `-sC`.

```bash
user@htb[/htb]$ nc -nv 10.129.42.253 21 
(UNKNOWN) [10.129.42.253] 21 (ftp) open 220 (vsFTPd 3.0.3)
```
> We've discovered the version of `vsFTPd`!

### SMB (Server Message Block)
`smb` is the protocol user for sharing files, printers and systems communications between machines (usually windows) within a network.
By default, it runs on **Port 445**. It allows machines to talk to each other and share access to folders, which are called **SMB shares**. 
See [[tools/smb|smb]] for more.

### SNMP (Simple Network Management Protocol)
SNMP is a network protocol that runs over **UDP Port 161** and is used to remotely monitor, manage and gather statistics from routers, witches, servers and printers. 
To read data from an SNMP server, you need a *community string*. 
1. `public`: Gives read-only access
2. `private`: Gives read/write access.
The default community strings of `public` and `private` are often left unchanged.
In SNMP versions 1 and 2c, access is controlled using a plaintext community string. If we know the name, we can gain access. Encryption and authentication were only added in SNMP version 3.

SNMP can be used to give us a lot of information:
- Examination of process parameters might reveal credentials passed on the commands line, which could be reused elsewhere.
- Routing information.
- Services bound to additional interfaces.
- Version of install software.

A tool such as [onesixtyone](https://github.com/trailofbits/onesixtyone) can be used to brute force the community string names using a dictionary file of common community strings such as the `dict.txt` file included in the GitHub repo for the tool.
```bash
user@htb[/htb]$ onesixtyone -c dict.txt 10.129.42.254 
Scanning 1 hosts, 51 communities 
10.129.42.254 [public] Linux gs-svcscan 5.4.0-66-generic #74-Ubuntu SMP Wed Jan 27 22:54:38 UTC 2021 x86_64
```

If we discover a valid string, like `public` in the above example, we can then use `snmpwalk` to dump the entire system database: 
```bash
snmpwalk -v2c -c public [TARGET_IP] 
``` 
- `-v2c` : Sets the SNMP protocol version to 2c (the most common lab setup). 
- `-c public` : Supplies the verified community string password.
