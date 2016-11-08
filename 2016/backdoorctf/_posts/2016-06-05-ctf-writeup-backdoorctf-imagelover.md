---
layout: post
title: "backdoorctf16 : Imagelover"
author: dade
date: 2016-06-05 13:37:00 -0700
slug: imagelover
category:
- 2016
- backdoorctf
---
**Category:** Web
**Points:** 70
**Description:**

> Find imagelover [here](http://hack.bckdr.in:6969/)<br />
Created by: Amanpreet Singh

# Investigation

We're told that admin will visit our page with the flag, so we just need to submit something that we control so that we can watch the traffic come to our server.

Being the fond Hackers (1995) fan that I am, I'll use my playground vps, gibson.zerocool.io to serve up a file that we can sniff traffic on. I'm using the gibson subdomain because I have ssl enforced on zerocool.io and wasn't sure if tshark would be able to read the content as it came in, or if it would just show up as ssl packets.

Instead of sniffing all our traffic though, let's get the IP address that imagelover will likely be coming from.

```
dig a hack.bckdr.in
```


# Solution

From here we now have what we need to fire up tshark and listen to the traffic coming into the server.

```
root@gibson:/home/dade# tshark -i eth0 -x host 188.166.184.216
```

Now that the sniffer is running, we submit http://gibson.zerocool.io to the imagelover input.

```
Running as user "root" and group "root". This could be dangerous.
Capturing on 'eth0'

[Packet Capture omitted. Do it yourself :)]

```

# Flag

This challenge is hosted permanently at [Backdoor](https://backdoor.sdslabs.co/challenges/IMAGELOVER), so go find the flag yourself!
