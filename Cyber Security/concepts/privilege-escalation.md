Once we gain access to a remote server, we're usually still a low-privileged user. Our next step is to find a way to escalate our privileges to `root` on Linux or `administrator`/`SYSTEM` on Windows.

## PrivEsc Checklists
We'll want to begin by thoroughly enumerating the box to find any potential vulnerabilities. We can find many checklists and cheat sheets online, such as  [HackTricks](https://book.hacktricks.xyz/) or [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings). 

Many of the commands above can be automatically run with a script to go through the report and look for weaknesses. We can run many scripts to automatically enumerate the box, some common Linux ones being  [LinEnum](https://github.com/rebootuser/LinEnum.git) and [linuxprivchecker](https://github.com/sleventyeleven/linuxprivchecker), and for Windows [Seatbelt](https://github.com/GhostPack/Seatbelt) and [JAWS](https://github.com/411Hall/JAWS).

Another useful tool is the [Privilege Escalation Awesome Scripts SUITE (PEASS)](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite), which has scripts for both and is well maintained.

> Note: These scripts will run many commands known for identifying vulnerabilities and create a lot of "noise" that may trigger anti-virus software or security monitoring software that looks for these types of events. This may prevent the scripts from running or even trigger an alarm that the system has been compromised. In some instances, we may want to do a manual enumeration instead of running scripts.

Let us take an example of running the Linux script from `PEASS` called `LinPEAS`:

```bash
$ ./linpeas.sh

...SNIP... 
Linux Privesc Checklist: https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist 
LEYEND: 
	RED/YELLOW: 99% a PE vector RED: 
	You must take a look at it 
	LightCyan: Users with console 
	Blue: Users without console & mounted devs 
	Green: Common things (users, groups, SUID/SGID, mounts, .sh scripts, cronjobs)     LightMangenta: Your username

====================================( Basic information)=====================================
OS: Linux version 3.9.0-73-generic
User & Groups: uid=33(www-data) gid=33(www-data) groups=33(www-data)
...SNIP...
```

Once the script runs, it collects information and displays it in an easy to parse report. Let's look at some vulnerabilities we should look out for from these scripts.

## Vulnerabilities
### Kernel Exploits
If we encounter a server running an old operating system, we should begin by looking for potential kernel vulnerabilities, as it is likely vulnerable to now patched exploits.

For example, the above script showed us the Linux version is `3.9.0-73-generic`. Searching for [[public-exploits]], we might find `CVE-2016-5195`, AKA [DirtyCow](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs), which would give us root access.

We should keep in mind kernel exploits can cause system instability and thus we should take care before running them on production systems. It's best practice to try them in a lab environment first, and only run them on production systems with express permission from and coordination with our client.

### Vulnerable Software
We should also look at installed software, for example with `dpkg -l` on Linux or by looking in `C:\Program Files` on Windows. Again, there could be public exploits for software in use, especially if any are out of date.

### User Privileges
Another critical aspect to look at are the privileges available to the user we have access to. We might have the ability to run specific commands as root or as another user, which we might be able to leverage to escalate or privileges, or gain access as another user.

Some common ways to exploit user privileges are:
1. [[Sudo]]
	1. We can search something like [GTFOBins](https://gtfobins.github.io/) for applications that we have `sudo` privilege over to see whether there exist any commands that might let us gain `root`.
2. SUID
3. Windows Token Privileges

[LOLBAS](https://lolbas-project.github.io/#) also contains a list of Windows applications that can be leveraged to perform certain functions, like downloading files or executing commands as a privileged user.

### Scheduled Tasks
Windows and Linux both have methods to run specific scripts at certain intervals, which we can take advantage of by:

1. Adding new scheduled tasks/cron jobs
2. Tricking them to run malicious software

The easiest way is to check whether we're allowed to add new scheduled tasks. In Linux, a common way to maintain scheduled tasks is through `Cron Jobs`. These are directories that we might be able to `write` new jobs to if we have the permissions. These include:

- `/etc/crontab`
- `/etc/cron.d`
- `/var/spool/cron/crontabs/root`

If we can write to a cron job directory, we can write a bash script with a reverse shell command.

### Exposed Credentials
We can look for files we can read and see if they contain any exposed credentials. This is especially common in `configuration`, `log` and user history (`bash_history` in Linux and `PSReadLine` in Windows) files. The enumeration scripts discussed earlier usually look for potential passwords in files and provide them for us:

```shell
...SNIP... 
[+] Searching passwords in config PHP files 
[+] Finding passwords inside logs (limit 70) 
...SNIP... 
/var/www/html/config.php: $conn = new mysqli(localhost, 'db_user', 'password123');
```

As we can see, the database password is exposed, which would allow us to log into the local mysql database and continue looking for anything interesting. We might also check for **password reuse**, as the user might've used the same password somewhere else:

```shell
$ su -
Password: password123

whoami
root
```

### SSH Keys
If we have read access over the `.ssh` directory for a specific user, we may be able to read their private ssh keys found in `/home/user/.ssh/id_rsa` or `/root/.ssh/id_rsa`, which we can use to log in to the server as them. We can copy it to our machine and use the `-i` flag to login with it:

```shell
$ vim id_rsa
$ chmod 600 id_rsa
$ ssh root@10.10.10.10 -i id_rsa

root@10.10.10.10#
```

> Note that we used the command 'chmod 600 id_rsa' on the key after we created it on our machine to change the file's permissions to be more restrictive. If ssh keys have lax permissions, i.e., maybe read by other people, the ssh server would prevent them from working.

If we find ourselves with the right to `write` to the `.ssh` directory, then we can place *our* public key in the `/home/user/.ssh/autorized_keys` directory. The current SSH configuration will not accept keys written by other users, so we must have gained a shell as that user. We must first create a new key with `ssh-keygen` and the `-f` flag to specify the output file:

```shell
$ ssh-keygen -f key 
Generating public/private rsa key pair. Enter passphrase (empty for no passphrase): ******* 
Enter same passphrase again: ******* 
Your identification has been saved in key Your public key has been saved in key.pub 
The key fingerprint is: SHA256:...SNIP... user@parrot 
The key's randomart image is: 
+---[RSA 3072]----+ 
| ..o.++.+ | 
...SNIP... 
| . ..oo+. | 
+----[SHA256]-----+
```

This gives us `key`, the private key we use `ssh -i` to log in with, and `key.pub`, the public key we'll place in `/root/.ssh/authorized_keys`:

```bash
user@remotehost$ echo "ssh-rsa AAAAB...SNIP...M= user@parrot" >> /root/.ssh/authorized_keys
```

Now, the remote server should allow us to log in as that user by using our private key:

```bash
$ ssh root@10.10.10.10 -i key  

root@remotehost#
```

We can now ssh in as `root`.
