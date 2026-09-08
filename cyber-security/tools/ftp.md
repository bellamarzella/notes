## What does it do
Connects to a remote server over Port 21 to browse, download or upload files. Used to look for leaked credentials, configuration files or leftover backup data. It is essentially a file explorer, all it can do is list, download or upload files.
## Usage
### Step 1: Connect to the server
Type the command followed by the target IP address:
```bash
ftp [TARGET_IP]
```
### Step 2: The Anonymous Login
When the terminal prompts you for credentials, check if the server is misconfigured by typing these exact inputs:
1. **Name:** `anonymous`
2. **Password:** *(Leave completely blank and hit Enter)*
If the screen says `230 Login successful`, you are inside their file system.
### Step 3: Extracting the Data
Once logged in, your terminal prompt changes to `ftp>`. Run these commands to hunt for files:
- `ls` : List all files and folders in the current directory.
- `cd [folder]` : Move into a hidden directory.
- `binary` : Type this *before* downloading any file to ensure images, zip files, or compiled scripts don't get corrupted during transfer.
- `get filename.txt` : Downloads the file directly from the target machine onto your local Kali machine.
- `exit` : Close the connection.
