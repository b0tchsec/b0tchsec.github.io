---
layout: post
title: "SHA2017 CTF - Junior : Zipfile One"
author: aagallag
date: 2017-08-06 15:18:00 -0700
slug: zipfileone
category:
- 2017
- sha2017
- sha2017-junior
---
**Category:** Misc

**Points:** 1

**Description:**

```
We received this zip file, but is asking for a password. All we know is that the password exists of 5 numbers, can you crack this password to get the hidden information?

zipfileone.zip - 8caeb32d9716898f9af223f9762c8b27
```

You are provided with a standard, password-protected zip file.  We are given enough information about the file that brute-forcing it will be rather quick.  I decided to use fcrackzip.

```
$ fcrackzip -c 1 -b -l 5-5 -u zipfileone.zip


PASSWORD FOUND!!!!: pw == 42831
```

Then simply unzip the file with the password `42831` to reveal a text file containing the flag.

# Flag
**flag{d6f56ae046bb241cc61f9d26f8e525d9}**
