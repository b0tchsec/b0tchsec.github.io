---
layout: post
title: "TU CTF 2016: The Nack"
date: 2016-05-20 06:42:06 -0700
comments: true
author: aagallag
slug: the-nack
category:
- 2016
- tuctf
---
**Category:** Forensic
**Points:** 100
**Description:**

> [Attached file]({{ site.static }}/2016/tuctf/the-nack/the_nack.pcapng)


# Investigate
I load up the capture file into Wireshark and start by checking for HTTP objects with...

```
File -> Export Objects -> HTTP...
```

Unfortunately, there aren't any HTTP objects found.

After checking for HTTP objects, my next step when working with packet captures is usually to inspect the individual TCP streams.

```
Right Click Packet -> Follow -> TCP Stream
```

<img src="{{ site.static }}/2016//tuctf/the-nack/tcp_stream_0_cropped.jpg" alt="TCP Stream 0" class="img-responsive"/>

Looks like the traffic may contain a GIF file, but why does it start with 'GOAT'?

<img src="{{ site.static }}/2016//tuctf/the-nack/tcp_stream_1_cropped.png" alt="TCP Stream 1" class="img-responsive"/>

Wireshark lets you quickly switch between the different streams by increasing the Stream counter at the bottom right of the pop-up window.  Scrolling through the first couple TCP streams reveals a pattern.

<img src="{{ site.static }}/2016//tuctf/the-nack/tcp_stream_15_cropped.png" alt="TCP Stream 15" class="img-responsive"/>

It looks like all of the streams start with the same 4 bytes, 'GOAT' followed by the byte 0x01.

# Code
Now that I know the pattern, I threw together a simple Python script that looks for all occurances of 'GOAT' followed by 0x01 and then writes the proceeding 4 bytes to a file.

```python
#!/usr/bin/env python

# read file contents into memory
f = open('ce6e1a612a1da91648306ace0cf7151e6531abc9.pcapng', 'rb')
content = f.read()
f.close()

#split on 'GOAT' + x01 byte (skipping the front part of file before GOAT starts)
goats = content.split('GOAT\x01')[1:]

#write the TCP data to a new file
f = open('goats.data', 'wb')
for i in goats:
	#data is in first 4 bytes, 5th byte should be null
	assert(i[4] == '\x00')
	data = i[:4]
	f.write(data)

f.close()

print('Goat data extracted...')
```

Next I wanted to confirm that my initial suspicion was correct about this being a .gif file.

```
$ file goats.data
goats.data: GIF image data, version 89a, 590 x 225
```

Great, let's see what the image looks like...

```
$ mv goats.data goats.gif
```

<img src="{{ site.static }}/2016//tuctf/the-nack/goats.gif" alt="goats.gif" class="img-responsive"/>

I don't see a flag there...  Maybe it's hiding in a single frame of the .gif and the flag is flashing too quickly for me to see it.  So let's extract each frame of the gif into seperate, non-moving images.

```
$ convert goats.gif out%05d.gif
$ ls out000*.gif
out00000.gif  out00005.gif  out00010.gif  out00015.gif  out00020.gif  out00025.gif
out00001.gif  out00006.gif  out00011.gif  out00016.gif  out00021.gif  out00026.gif
out00002.gif  out00007.gif  out00012.gif  out00017.gif  out00022.gif  out00027.gif
out00003.gif  out00008.gif  out00013.gif  out00018.gif  out00023.gif  out00028.gif
out00004.gif  out00009.gif  out00014.gif  out00019.gif  out00024.gif
```

I check each frame of the gif, the 17th frame(out00016.gif) reveals something interesting...

<img src="{{ site.static }}/2016//tuctf/the-nack/out00016.gif" alt="out00016.gif" class="img-responsive"/>

# Flag

```
TUCTF{this_transport_layer_is_a_syn}
```
