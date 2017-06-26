---
layout: post
title: "TJCTF 2016: Torrent"
date: 2016-05-31 11:12:45 -0700
comments: true
author: aagallag
slug: torrent
category:
- 2016
- tjctf
---
**Category:** misc
**Points:** 90
**Description:**

> Help, someone's sharing [flags](https://github.com/b0tchsec/CTF-Fanny-Pack/blob/master/solutions/tjctf_2016/torrent/flag_d542574c3e963792fe07321fd262fe28e6f5cf0ea126efe01148dd6890b63a4d.torrent).


# Investigate
For those of you who may be living under a rock and have never heard of BitTorrent, I recommend skimming through the [BitTorrent](https://en.wikipedia.org/wiki/BitTorrent) and [Torrent file](https://en.wikipedia.org/wiki/Torrent_file) wiki pages. But to give a brief overview, torrent files don't actually contain the desired file.  But rather, they provide information to a Torrent client about how to download the desired file from peers on the internet.  But how does your torrent client find suitable peers?  Traditionally, the .torrent file contains the address to a torrent tracker that keeps track of who is downloading the file and who is currently sharing the file.

So I started by loading up the file into my torrent application to check and see if it will download anything.

<img src="{{ site.static }}/2016/tjctf/torrent/1_torrentclient.png" class="img-responsive"/>

Hmm... Doesn't seem like the torrent is downloading anything, let's poke around the torrent properties to see why.

Let's start by checking out the trackers.

<img src="{{ site.static }}/2016/tjctf/torrent/2_trackers.png" class="img-responsive"/>

Oh great, there are no trackers.  How can the file be downloaded without any trackers?  It turns out there have been new protocols applied ontop of torrents to allow peer discovery without trackers, checkout the [DHT](https://en.wikipedia.org/wiki/Mainline_DHT) and [PEX](https://en.wikipedia.org/wiki/Peer_exchange) wiki pages for more information on these protocols.

Perhaps, to solve this challenge, we have to find the tracker or peer hosting the file. **SPOILER ALERT:  this was not the correct approach.**

By default, I like to keep the peer discovery protocols disabled due to privacy concerns (for those concerned with privacy, you probably shouldn't be using BitTorrent in the first place).  So for the duration of this challenge, I went ahead and enabled them.

<img src="{{ site.static }}/2016/tjctf/torrent/3_dht_pex_settings.png" class="img-responsive"/>

Hmm... Still, my client doesn't download the file.  So I checked out the peerlist.  After running nmap on some of the peers, I noticed one of them was hosting a torrent tracker.  So I tried to set that peer as my tracker, but still, the file didn't download.  After noticing all these peers were hosting different services, and running different OS's, I realized these peers were unlikely to be hosting the flag, but instead, I'm pretty sure they were other participants in the CTF.

Oops... before I get kicked out of the CTF for hacking other contestents, I decided to take a step back and stop attempting to download the file altogether.

Let's see what information the *.torrent file reveals about the file.

<img src="{{ site.static }}/2016/tjctf/torrent/4_torrent_properties.png" class="img-responsive"/>

Hmm, looks like *.torrent file contains the SHA1 hash of the file.  This is required by the Torrent client so that once it has finished downloading the file, it can verify the integrity by generating a hash of the downloaded file and comparing it with the expected hash.

What if I try and work backwards from the hash?  I could pass a bunch of random data into a hash function until it computes to the same hash.  Nope, that would take too much time for any unique string larger than a few bytes.

Good thing I know a little bit about the BitTorrent protocol.  My client doesn't show it, but in addition to the entire file, the sha1-hash for each piece is also stored in the *.torrent file.  That way, in the event that your client detects that the downloaded file is corrupt, it doesn't have to throw out everything and start from scratch.  Instead, your client checks the hash of each piece and re-downloads only the piece(s) that failed.

Using this knowledge, I realized that if the pieces are small enough, I could resolve the hash back to the piece a lot quicker than attempting to crack the hash for the entire file.

However, my Torrent client isn't showing me enough information, I need to find a way to extract more details.

I found this cool Python project that parses torrent files and generates an HTML report, called [bittorrent-parser](https://github.com/Resistor52/bittorrent-parser).

To setup and run the tool...

```bash
$ git clone https://github.com/Resistor52/bittorrent-parser.git
$ python ./bittorrent-parser/parse-torrent.py flag_d542574c3e963792fe07321fd262fe28e6f5cf0ea126efe01148dd6890b63a4d.torrent 
```

After runnning the tool, I load up output.html into a web browser.

<img src="{{ site.static }}/2016/tjctf/torrent/5_output_html.png" class="img-responsive"/>

Wow, I wish this had been my first step, this parser is a lot more precise and verbose than your average torrent client.

I notice that the announce url (torrent tracker) was set to 127.0.0.1, that explains why the file could not be downloaded.  Next I see that the exact size of the flag file is only 28 bytes with 14 pieces.  Each piece is only 2bytes long!  That's great, cracking the hash for a string whose length we know to only be 2 is WAY quicker than attempting to crack the hash for the entire file.

So now I have to simply figure out which two bytes results in each SHA1-hash.  Once I have each 2byte piece, I simply put them together to recreate the original file.  Next section will show how I went about doing that.

# Code
Since I know the file is only 28 bytes long, I predict that this file doesn't contain any binary data.  So I make the assumption that this is a plaintext ASCII encoded file.  So to speedup the cracking process, I only search for values in the ASCII range of 0->126.  There are additional modifications that I could have made to ensure that the cracking process would be fast, but this implementation was sufficient for solving the challenge.

```python
#!/usr/bin/env python
import sha

# hashes for each piece of the torrent file
PIECE_HASHES = [
'546b05909706652891a87f7bfe385ae147f61f91',
'589e942e00a7dd64a273deb5041c7ce469f2bad7',
'b411d7823a3c4ee3773cafca1e36b8cfd26655ba',
'f437cb078acc7c6d79873462334a355eddeb9459',
'b504c843b2ef4c55c673be0b1daf3b12c5cf2fe8',
'0699989c219e1d7b336851c646e88a651859d081',
'd8273e2f4a7c0a59554544c6605cdd8b117848aa',
'f2daf7bf8c0100e8421f6a72dd8064cad674813a',
'd0cf1ef21f0ce65584e2453a3fb427f6591adca8',
'1116ef128bb637e2d69e9666bfe6d8a4ef9d2c13',
'408c28a2da80ef8bc57e580ac9ffc7f69b2a0e0e',
'f9fc27b9374ad1e3bf34fdbcec3a4fd632427fed',
'c387c982a132d05cbd5f88840aef2c8157740049',
'b3127725f678ca5b1038b1df45a06f2ff4e1f544',
]


def crack(shahash):
    for x in range(127):
        for y in range(127):
            attempt = chr(x) + chr(y)
            res = sha.new(attempt).hexdigest()
            if res == shahash:
                return attempt

    raise Exception('Failed to crack the hash: ' + shahash)


def main():
    flag = ''
    for i in PIECE_HASHES:
        flag += crack(i)
    print(flag)


if __name__ == "__main__":
    main()

```


# Flag
Run the script to get the flag.

```bash
$ ./torrent_hash_cracker.py 
tjctf{pls_2_n0t_fl4g_share}

```
