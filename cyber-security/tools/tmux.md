## What does it do
Allows you to split a single terminal window into multiple panes. Essential for keeping OpenVPN connection running in one pane, an Nmap scan in another, and a Netcat listener or tool window in a third, for example.
## Usage
### Step 0: Command Line Setup
```bash
tmux          # Start a new tmux session
tmux attach   # Reconnect to my session if the window accidentally closed
```
### Step 1: Execute Shortcuts (The Prefix Combo)
To run any Tmux command, you must always press **`Ctrl + B` first**, release it, and then hit the specific shortcut key.
### Panes (Splitting the Screen)
- `Ctrl + B` then `%` : Split the terminal vertically (Left / Right).
- `Ctrl + B` then `"` : Split the terminal horizontally (Top / Bottom).
- `Ctrl + B` then `Arrow Keys` : Move your cursor between the split panes.
- `Ctrl + D` or type `exit` : Close the current active pane.
### Windows (Tabs)
- `Ctrl + B` then `c` : Create a brand new full-screen window (tab).
- `Ctrl + B` then `n` : Switch to the next window.
- `Ctrl + B` then `p` : Switch to the previous window.
