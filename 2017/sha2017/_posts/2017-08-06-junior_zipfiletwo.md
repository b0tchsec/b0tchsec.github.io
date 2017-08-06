---
layout: post
title: "SHA2017 CTF - Junior : Zipfile Two"
author: aagallag
date: 2017-08-06 15:56:00 -0700
slug: zipfiletwo
category:
- 2017
- sha2017
- sha2017-junior
---
**Category:** Misc

**Points:** 2

**Description:**

```
We received another zip file, which also requires a password. All we know is that the password is an existing English word with a length of 6 and all lowercase. Can you crack this password?

zipfiletwo.zip - 72bac30689c07b30cf9a4c6f74bcbdd9
```

Very similar to Zipfile Two, we are provided with enough information about the file that brute-forcing it will be rather quick.  Again, I used fcrackzip but tweaked the parameters to match this challenge.

```
$ fcrackzip -c a -b -l 6-6 -u zipfiletwo.zip


PASSWORD FOUND!!!!: pw == future
```

And again, unzip the file with the password `future` to reveal a text file containing the flag.

# Flag
**flag{7128d78caf1e3297386a09afae0f8ea4}**
