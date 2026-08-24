## What does it do?
The `sudo` command allows a user to **execute commands as a different user.** It is usually used to allow lower privileged users to execute specific commands as `root` without giving them access to the `root` user.
This is generally done as some useful commands, such as `tcpdump`, can only be run as root, or to allow the user access to certain root-only directories. 

## Usage

**We can check `sudo` privileges we have access to with the `sudo -l` command**:

```bash
$ sudo -l

[sudo] password for user1:
...SNIP...
User user1 may run the following commands on ExampleServer: 
	(ALL : ALL) ALL
```

We can see that we're allowed to run all commands with `sudo`, so we can use `sudo su` to switch to the `root` user:

```bash
$ sudo su -

[sudo] password for user1: 
$ whoami 
root
```

The above commands, however, require a password. There are certain occasions in which we can execute commands or applications without a password:

```bash
$ sudo -l

```