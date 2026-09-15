### Python

1. Python Scripts:
#### Python
```python
python -c 'import pty; pty.spawn("/bin/bash")'
```
#### Python 3
```python
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

2. Send the  `nc` session to the background with `ctrl + z`.
3. In the local terminal, run `stty raw -echo; fg`
4. Hit `enter` twice, then reset environment variables with `export TERM=xterm-256color`

