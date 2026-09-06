
## What does it do 
Connects to Windows Server Message Block (SMB) shares on Port 445 to view or download shared files and network resources.
## Usage
### Step 1: List the available shares
Check if the server lets you view folder names without a password by running:
```bash
smbclient -L '\\[target ip]\' -N
```
- `-L` : Lists the available shares (folders) on the target.
- `-N` : No password. Tells the tool to attempt a null session connection.
### Step 2: Connect to a specific share
If Step 1 shows an interesting folder name (like `\backups` or `\shares`), connect straight to it:
```bash
smbclient \\[target ip]\[share name] -N
```
*(Once inside, you use the exact same commands as [[ftp]]: `ls` to look around, and `get filename.txt` to download files).*
