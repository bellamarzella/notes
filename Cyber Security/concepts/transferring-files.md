We'll often need to transfer files between the attacker and victim.

## wget
One method is to run a [Python HTTP server](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/set_up_a_local_testing_server) on our machine and use `wget` or `curl` to download the file on the remote host. First, `cd` to the file we need to transfer and run a `Python HTTP` server in it:
```
```