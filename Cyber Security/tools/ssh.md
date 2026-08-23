## What does it do
Basically a legitimate bind-shell. Allows us to connect to a server and run commands as if it were our own machine.
## Usage
```bash
ssh [username]@[hostname or IP address]
```
## Flags
- `-p [port number]`: If `SSH` isn't the default 22, we can specify which port we want to connect to.
- `-i [path to key file]`: If the server verifies the user with a private RSA key instead of a password, we can specify the file.
