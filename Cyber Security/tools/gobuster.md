## What does it do
A high-speed brute-force fuzzer used to discover hidden assets on a target. It takes a wordlist and tries them sequentially against a server to find assets that are not linked anywhere publicly.
## Usage
### Step 0: Syntax
```bash
gobuster [mode] -u [target url] -w [wordlist path] [flags]
```
### Step 1: Directory Scan
Scans a website to map out hidden folders and administration panels:
```bash
gobuster dir -u http://[target ip:[port] -w /usr/share/seclists/Discovery/Web-Content/common.txt
```
> [Seclists](https://github.com/danielmiessler/seclists)  is a repo full of wordlists.
## Flags
### Modes
- `dir`: Directs Gobuster to run in Directory and File enumeration mode.
- `dns`: Directs Gobuster to brute-force subdomains instead of web folders.
### Core Configuration
- `-u [URL]`: The target URL. Must include the protocol (`http://` or `https://`) and the custom port number if it isn't standard port 80.
- `-w [PATH]`: The local file path to the text wordlist you want to use.
- `-x [EXTENSIONS]`: Appends file formats to the words (e.g., `-x php` turns the word `config` into `config.php`).
### Performance & Output
- `-t n`: Sets the number of concurrent threads to $n$ (Default is 10. Use 50 or 100 on HTB to speed up the scan).
- `-s [CODES]`: Filters the terminal output to only display specific HTTP status codes (e.g., `-s "200,403"`).
