Once we gain access to a remote server, we're usually still a low-privileged user. Our next step is to find a way to escalate our privileges to `root` on Linux or `administrator`/`SYSTEM` on Windows.

## PrivEsc Checklists
We'll want to begin by thoroughly enumerating the box to find any potential vulnerabilities. We can find many checklists and cheat sheets online, such as  [HackTricks](https://book.hacktricks.xyz/) or [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings). 

Many of the commands above can be automatically run with a script to go through the report and look for weaknesses. We can run many scripts to automatically enumerate the box, some common Linux ones being  [LinEnum](https://github.com/rebootuser/LinEnum.git) and [linuxprivchecker](https://github.com/sleventyeleven/linuxprivchecker), and for Windows [Seatbelt](https://github.com/GhostPack/Seatbelt) and [JAWS](https://github.com/411Hall/JAWS).

Another useful tool is the 