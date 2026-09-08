## What does it do
A text editor that runs completely inside the terminal. Used for modifying exploit scripts, writing quick text files, or cleaning up wordlists directly on the command line.
## Usage
### Step 1: Open a file
```bash
vim filename.txt
```
### Step 2: Navigate the 2 Modes
Vim starts in **Normal Mode** by default. You cannot type text here, only run navigation shortcuts. You must manually swap modes to edit.
1. Press `i` to enter **Insert Mode** (now you can type and edit text normally).
2. Press `Esc` to exit back to **Normal Mode** when you are finished typing.
### Step 3: Saving and Quitting
You must be in **Normal Mode**. Type a colon `:` followed by your instruction, then hit enter:
- `:w`  : Write (Save) the changes to the disk.
- `:q!` : Quit immediately and discard any unsaved changes.
- `:wq` : Save changes and quit out of the file.
## Shortcuts (Normal Mode Only)
- `dd` : Delete (cut) the entire current line.
- `yy` : Copy (yank) the current line.
- `p`  : Paste the copied or cut line down below the cursor.
- `/keyword` : Search the file for a specific word. Press `n` to jump to the next match.
