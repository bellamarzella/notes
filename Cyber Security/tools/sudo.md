## What does it do?
The `sudo` command allows a user to **execute commands as a different user.** It is usually used to allow lower privileged users to execute specific commands as `root` without giving them access to the `root` user.
This is generally done as some useful commands, such as `tcpdump`, can only be run as root, or to allow the user access to certain root-only directories. 

**We can check `sudo` privileges we have access to with the `sudo -l` command**:
```bash
$ sudo -l


```