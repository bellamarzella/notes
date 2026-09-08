## What does it do
Connects your machine to an isolated network. Without this running, you cannot reach, ping, or scan any internal target IP addresses.
## Usage
### Step 1: Run the connection
Navigate to the directory where you downloaded your configuration file and run it with root privileges:
```bash
sudo openvpn [user.ovpn]
```
### Step 2: Verify the connection
Open a new terminal tab and check for an active lab IP address assigned to the virtual interface:
```bash
ip a s tun0
```

## Flags & Mechanics
- `sudo`: Required because OpenVPN must have root permissions to create a virtual network interface (`tun0`) on my local machine.
- **Connection Drops:** If the connection freezes or drops, press `Ctrl + C` to kill the process and execute the command again. You must leave this terminal pane or window open the entire time you are hacking the box.
