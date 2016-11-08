---
layout: post
title: "ASIS CTF Quals 2016: firtog"
date: 2016-05-10 08:59:06 -0700
comments: true
author: aaron
slug: firtog
category:
- 2016
- asis
---
**Category:** Forensic
**Points:** 109
**Solves:** 113
**Description:**

> [Obscurity](https://github.com/b0tchsec/CTF-Fanny-Pack/blob/f0b9c2aa8c2be4c4fbcc7cf381ffa1a8d55249b1/solutions/asis_2016_quals/firtog/firtog_75813ce76bbf9014bb7afae8071e180e0b939d31713062baf3ffafd852f1f7687e8d1cf61762bb80245adf3fec5cbcf01e3e746bbaf3a880fd13f61b122080c4) is definitely not security.


# Investigate
As with the other challenges in this CTF, I start by renaming the file to *.tar.xz and extract the contents.  Once extracted, I find a TCP packet capture file called firtog.pcap.  I loaded up the capture file into Wireshark and started by checking for HTTP objects with...

```
File -> Export Objects -> HTTP...
```

Unfortunately, there aren't any HTTP objects found.

After checking for HTTP objects, my next step when working with packet captures is usually to inspect the individual TCP streams.

```
Right Click Packet -> Follow -> TCP Stream
```

Wireshark lets you quickly switch between the different streams by increasing the Stream counter at the bottom right of the pop-up window.  Scrolling through the first couple TCP streams reveals that this is Git traffic.

Now that we know we're looking at Git traffic, how do we extract the data?  There may be cleaner ways of doing it, but I decide to manually dump the raw bytes from each TCP stream.  Wireshark makes this really easy.  With the 'Follow TCP Stream' window open, select only the direction of traffic you would like to capture.  In the case of Git, we're interested in the server's traffic.  By default, Wireshark encodes the data as ASCII, change this to 'Raw' and click the 'Save as...' button.  I did this for each TCP stream.

Then I ran the following command to extract the data from all TCP streams at once.

```
$ binwalk -e *
```

This ended up revealing multiple revisions of the Python script used to generate and encode the flag.  Using the commit messages (and a little bit of intuition) I realized that this was the final revision of the script.


```python
#!/usr/bin/python
# Simple but secure flag generator for ASIS CTF

from os import urandom
from hashlib import md5

l = 128
rd = urandom(l)
h = md5(rd).hexdigest()
flag = 'ASIS{' + h + '}'
f = open('flag.txt', 'r').read()
flag = ''
for c in f:
	code = hex(pow(ord(c), 65537, 143))[2:]
	print('%s => %s' % (c,code))
	flag += code
print flag
```

And additionally, the encoded flag was also checked into the Git repo.

```
41608a606a63201245f1020d205f1612147463d85d125c1416635c854c74d172010105c14f8555d125c3c
```

Now my mission is clear!  I have to reverse the encoded flag back to the original flag text.

# Code
It looks like this is the code we want to crack...

```
code = hex(pow(ord(c), 65537, 143))[2:]
```

Since this isn't a real encryption algorithm, there isn't any chaining involved.  That means that each character of the original message is encoded independently of each other.  This makes it very easy to crack!

My approach was to simply pre-calculate the value of all printable ASCII characters and store these in a dictionary.  Then to decode a given message, we just lookup the encoded value (key) in the dictionary and it should return the non-encoded value.  No different than a decoder ring on the back of a cereal box.

<img src="{{ site.static }}/2016/asis/firtog/christmas-story3.jpg" alt="Be sure to drink your ovaltine" class="img-responsive"/>

So first we must build the decoder ring.  Notice, I only chose to pre-calculate values 32-126, as this is the range for printable ASCII characters.  If I included too many numbers to pre-calculate, we run the risk of experiencing collisions (2 different plain-text characters could get encoded to the same value).

```python
def build_decoder_ring():
	decoder_ring = {}
	for c in range(32,126):
		code = hex(pow(c, 65537, 143))[2:]
		decoder_ring[code] = chr(c)
	return decoder_ring
```

My first attempt at decoding the flag had a bug though.  The code looked like this...

```python
def crack_encoded_msg(encmsg):
	decoder_ring = build_decoder_ring()

	flag = ''
	i = 0
	while (i < len(encmsg)):
		flag += find_possible_matches(decoder_ring, encmsg[i] + encmsg[i+1])
		i += 2
	return flag
```

The problem was that not all characters got encoded as 2 hex characters.  Some get encoded as just a single hex character.  So I tweaked the code to perform a try/catch.  If it fails to find a match on 2 characters, it looks for a match on just the first character.

My final solution:

```python
#!/usr/bin/env python


def build_decoder_ring():
	decoder_ring = {}
	for c in range(32,126):
		code = hex(pow(c, 65537, 143))[2:]
		decoder_ring[code] = chr(c)
	return decoder_ring


def find_possible_matches(decoder_ring, two_char):
	return decoder_ring[two_char]


def crack_encoded_msg(encmsg):
	decoder_ring = build_decoder_ring()

	flag = ''
	i = 0
	while (i < len(encmsg)):
		try:
			flag += find_possible_matches(decoder_ring, encmsg[i] + encmsg[i+1])
			i += 2
		except:
			flag += find_possible_matches(decoder_ring, encmsg[i])
			i += 1
	return flag


def main():
	encflag = open('flag.txt', 'rb').read()
	flag = crack_encoded_msg(encflag)
	print(flag)


if __name__ == "__main__":
	main()
```

# Flag

```
ASIS{c691a0646e79f3c4d495f7c5db3486005fad2495}
```
