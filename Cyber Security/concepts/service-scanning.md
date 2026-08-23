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

