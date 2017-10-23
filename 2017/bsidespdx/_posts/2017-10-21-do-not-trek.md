---
layout: post
title: "BSidesPDX CTF : DoNotTrek"
author: dade
date: 2017-10-21 13:37:00 -0700
slug: do-not-trek
category:
- 2017
- bsidespdx
---
**Category:** Web

**Points:** 100

**Description:**

```
To boldly go where trackers may or may not track you. Also have you seen my pet snake?

host: a9a4a876db2d711e7887102abc71843c-387472393.eu-central-1.elb.amazonaws.com

```

# Note
The BSidesPDX organizers have made the source code for all of their challenges freely available so that you can run them at home and follow along. You can find more information [here](https://github.com/BSidesPDX/CTF-2017).

# Investigation
Upon loading the host, we see a flying spaceship Enterprise, with some marqueeing text that says "Do not track is 0!" Since Do Not Track is a common HTTP header you can send websites to ask not to be tracked, we know the web app must be reflecting our value back to us in some regard. The first step I took was to see if I could evaluate math in the `DNT` header. I fired up burp and set it to intercept requests, then reloaded the page. 

When the intercept came up, I changed `DNT: 0` to `DNT: 1+1`. Lo and behold, the page reflected `Do not track is 2!` Next I tried to inject some other stuff and see what else was allowed. By modifying the DNT header to read `DNT: globals()` I was able to confirm that python was getting evaluated in the header.

From here I decided to have a look at the filesystem so I intercepted another request and set it to `DNT: __builtins__.__import__('subprocess').check_output(["ls", "-la"])` which yielded the contents of the current directory into the page. From here I could see that there was a file named flag.

# Solution
Create or intercept an http request to the challenge host and set the DNT flag to:
`DNT: __builtins__.__import__('subprocess').check_output(["cat", "flag"])`

# Flag
**BSidesPDX{d0_tr4ckers_3ven_EVAL_th1s_he4d3r}**
