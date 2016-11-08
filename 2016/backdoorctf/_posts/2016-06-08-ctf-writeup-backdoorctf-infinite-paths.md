---
layout: post
title: "backdoorctf16 : Infinite Paths"
author: dade
date: 2016-06-07 13:37:00 -0700
slug: infinite-paths
category:
- 2016
- backdoorctf
---
**Category:** Web
**Points:** 100
**Description:**

>cr4wl3r(a basillisk) once got pissed off with feignix(fawkes the feignix) and challenged him to find the flag that was hidden in the mysterious tunnels inside his lair, the chamber of secrets. feignix now flies inside the underground tunnels attempting to find the flag. See if you(Tom) can get to the flag first with some magic tricks. Will you be able to solve this Marvelous Riddle Tom? Go [here](http://hack.bckdr.in/INFINITE-PATHS/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path/path)<br />
Created by: Arpit Singla

# Note
I stumbled into a bug with the problem while trying to solve this that enabled me to completely bypass the intended solution. The contents below are how the problem was meant to be solved.

# Investigation

We're chucked into this path and simply told to keep roaming. Directory traversal attacks can be useful, except usually they don't come right out and say "change directories." Since we're told to keep roaming, let's try a few quick options. Let's add another `/path` to the end of the url.

```
Want the flag? Here it is:
"Hunh! D0 y0u 7h1nk 17'5 7h1s e4sy?"
```

Surely that's not the flag, it's taunting us that it couldn't be this easy. Classic red herring (The guys at SDSLabs really liked the red herrings this year). Let's also check out simply `http://hack.bckdr.in/INFINITE-PATHS/`. We're presented with the same taunting message. I'd be lying if I said I didn't go through and check almost every /path between 0 and 50 (the number of /path's we're presented with when the challenge begins). No luck traversing through all those directories, I suppose it's a good time to pull up [fiddler](http://www.telerik.com/fiddler), though you could also use burp suite if you have it setup.

Once we're monitoring fiddler, I opened up a clean browser instance so that I could mimic the very first interaction I had with the site.

<img src="{{ site.static }}/2016/backdoorctf/infinite-paths/01-fiddler.png" class="img-responsive" style="border:1px dashed red"/>

Here we can see a ton of cookies are set. Fitting, since we're a ton of directories into the site. Let's send another request so that we can clone a request without having to manually transform those Set-Cookie headers into a format suitable for a GET request. 

<img src="{{ site.static }}/2016/backdoorctf/infinite-paths/02-fiddler.png" class="img-responsive" style="border:1px dashed red"/>

Now let's go to the raw tab and copy the entire request we just sent so that we can do some modifying. Copy the request and then go to the Composer tab and paste it in. Be sure to have one blank line at the end of your request, otherwise it's not a valid request. Let's start off by removing the first cookie we see on the list and see what happens.

<img src="{{ site.static }}/2016/backdoorctf/infinite-paths/03-fiddler.png" class="img-responsive" style="border:1px dashed red"/>

Fascinating! We've gained useful information here. Let's try to remove another cookie from the front of the list and see what happens.

<img src="{{ site.static }}/2016/backdoorctf/infinite-paths/04-fiddler.png" class="img-responsive" style="border:1px dashed red"/>

Of course, that would have been too easy. Since there were a ton of /path's in the url, and a ton of cookies, let's count them both and see how they relate. I simply pasted the entire get request into sublime text and did a find on `/path` to get a count, and a find on `infinite_paths=` to get a count of the cookies. In our original request we had 51 cookies and only 50 paths.

# Solution

Now that we know there were originally 51 cookies being set at once and 50 paths, and that when we removed one cookie such that count(cookies) == count(paths), we were presented with one character of a 50 character flag. To confirm this we'll simply try to remove the first two cookies again, and this time also remove one of the /paths. 


We're sending 49 paths, and ~~49 cookies~~ we're actually only sending the first `infinite_paths` cookie on our list, since they all have the same name. I just kept the entire list of them handy and continued to pop one off the front in order to keep my place.

<img src="{{ site.static }}/2016/backdoorctf/infinite-paths/05-fiddler.png" class="img-responsive" style="border:1px dashed red"/>

Fantastic, our suspicion is confirmed and we can move on. You simply need to iterate through all 50 characters, popping one cookie off the front of the list every time. When we've finished we reach the last character of the flag.

<img src="{{ site.static }}/2016/backdoorctf/infinite-paths/06-fiddler.png" class="img-responsive" style="border:1px dashed red"/>

When you put all the characters you're presented with together, you should realize that it doesn't really make much sense. It begins with a period and ends with a capital letter. Let's go ahead and reverse that, which you can quickly do in python like so:

```
python -c 'print "TheFlagYouFind"[::-1]'
```

# My Accidental Solution

During some testing, I accidentally sent a `GET` request that contained %20 at the end of the request, immediately following the final `/path`. This presented me with a character of the flag. Upon a bit more investigation, I was able to obtain every character of the flag by simply putting something unexpected at the end of the string for every iteration of the `/path` list, completely bypassing the cookie challenge. This was not the intended solution and I informed the author.

```
http://hack.bckdr.in/INFINITE-PATHS/pathHack
http://hack.bckdr.in/INFINITE-PATHS/path/pathThe
http://hack.bckdr.in/INFINITE-PATHS/path/path/pathPlanet
...
```

# Flag

This challenge is hosted permanently at [Backdoor](https://backdoor.sdslabs.co/challenges/INFINITE-PATHS), so go find the flag yourself!
