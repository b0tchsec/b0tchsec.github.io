---
layout: post
title: "backdoorctf16 : Busybee"
author: dade
date: 2016-06-08 02:00:00 -0700
slug: busybee
category:
- 2016
- backdoorctf
---
**Category:** Forensics
**Points:** 150
**Description:**

>A deadly virus is killing bees in Busybee's village Busybox, India. Unfortuantely, you have to go to the village to fight the infection. Get the flag virus out of the infected files. <br />
Village address: [http://hack.bckdr.in/BUSYBEE/infected.tar](http://hack.bckdr.in/BUSYBEE/infected.tar)
<br />
Created by: Ashish Chaudhary

# Investigation
Let's unpack our infected.tar and take a look around

```
drwxr-xr-x  3 dade dade 4096 Jun  4 12:50 2b0fbc0e1ac044737fd881cff8164bb5a2c7bfbf90c40c87de3c3435f2c6a94e
drwxr-xr-x  4 dade dade 4096 Jun  4 12:53 983179bdb58ea980ec1fe7c45f63571d49b140bdd629f234be9c00a6edd8a4a7
drwxr-xr-x 10 dade dade 4096 Jun  4 12:51 d51a083a3b01fe8c58086903595b91fc975de59a9e9ececec755df384a181026
-rw-r--r--  1 dade dade 1344 Jun  3 09:13 eaa21323de5e2cce7078df3af4dd292181114dfc94be761b948657efbe3af26b.json
-rw-r--r--  1 dade dade  373 Dec 31  1969 manifest.json
-rw-r--r--  1 dade dade  106 Dec 31  1969 repositories
```

The json files don't really have much interesting in them, looks primarily like data from building the problem. Lets dig in to 2b0fbc0e...

```
dade@gibson:~$ ls -la infected/2b0fbc0e1ac044737fd881cff8164bb5a2c7bfbf90c40c87de3c3435f2c6a94e
-rw-r--r-- 1 dade dade  464 Jun  3 09:13 json
-rw-r--r-- 1 dade dade 1024 Jun  3 09:13 layer.tar
-rw-r--r-- 1 dade dade    3 Jun  3 09:13 VERSION
```

`file layer.tar` says "data" and `tar xvf layer.tar` is unable to get anything. `xxd layer.tar` shows us that the data is all 0s. Nothing of interest in json or VERSION

Let's move on to 983179bd...

```
-rw-r--r-- 1 dade dade     907 Jun  3 09:13 json
-rw-r--r-- 1 dade dade 1035776 Jun  3 09:13 layer.tar
-rw-r--r-- 1 dade dade       3 Jun  3 09:13 VERSION
```

json and VERSION are probably still not particularly helpful for us, but lets unpack this layer.tar

```
dade@gibson:~/infected/983179bdb58ea980ec1fe7c45f63571d49b140bdd629f234be9c00a6edd8a4a7$ tar xvf layer.tar 
bin/
bin/cat
bin/sha1sum
root/
root/.ash_history
```

`.ash_history` looks like a shell history file, let's see if anything interesting is in there.

```
dade@gibson:~/infected/983179bdb58ea980ec1fe7c45f63571d49b140bdd629f234be9c00a6edd8a4a7$ cat root/.ash_history
not so easy bru - the infections is intense
```

# Solution 

Hmm. This tarball also had two binaries in it. Let's see if there is a reason those binaries specifically were included by checking them out with strings. Given the premise of the challenge, that the flag had "infected" files by a "virus", binaries are a common target to hide in.

```
dade@gibson:~/infected/983179bdb58ea980ec1fe7c45f63571d49b140bdd629f234be9c00a6edd8a4a7$ strings bin/sha1sum 
[...]
THIS IS WHAT YOU ARE LOOKING FOR:    [REDACTED]
```

Well that was easy, the virus flag was in the infected binaries. Just sha256 that string and submit.

# Flag

This challenge is hosted permanently at [Backdoor](https://backdoor.sdslabs.co/challenges/BUSYBEE), so go find the flag yourself!

