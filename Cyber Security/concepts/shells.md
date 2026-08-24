A `shell` is a *program that takes input from the user via the keyboard and passes these commands to the operating system*. Practically speaking, it's **just the terminal command line.**
To **obtain a shell** means that we have successfully **forced the target machine to give us a remote CLI**. 
There are three types of shells:
## Reverse Shell
#### Description
An attack where the victim computer initiates the network connection back to the attacker’s machine, handing over its command line terminal.
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
