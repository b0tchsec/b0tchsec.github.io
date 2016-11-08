---
layout: post
title: "backdoorctf16 : CLUE"
author: dade
date: 2016-06-05 13:57:00 -0700
slug: clue
category:
- 2016
- backdoorctf
---
**Category:** Web
**Points:** 200
**Description:**

>Vampire has started recruiting hackers for his new team. To filter people, he has given a clue somehwhere [here](http://hack.bckdr.in/CLUE/). If you think you are capable enough to join him find the flag and submit its SHA-256 hash.
<br />Created by: Dhaval Kapil

#### This challenge was supposed to be eas(y|ier) but only two teams managed to solve it. b0tch_sec was team #2.

# Investigation
Upon visiting the site, we're told a flag is somewhere in this directory. Let's see what other things are in this directory, maybe it'll help.

My first instinct is to look for a `robots.txt`, which doesn't pan out. Then I look for `.htaccess`, which also doesn't pan out. Next I want to see if a .git folder is in available.

403 Forbidden. Bingo, we have a .git directory. It's not indexable though, so we're going to have to look for common files manually. My preferred first file to grab is always `.git/logs/HEAD` as it has the potential to provide a ton of information.

Once confirming we can access http://hack.bckdr.in/CLUE/.git/logs/HEAD I want to go ahead and rip the entire git directory. I originally did this by manually building a couple tools I could use to download every object, view the contents of the object, then download any objects referenced from there. But a little searching around left me with [rip-git](https://github.com/kost/dvcs-ripper/blob/master/rip-git.pl) which is way more useful. Let's go ahead and run that with the url we know to have .git. 

```
rip-git.pl -v -u http://hack.bckdr.in/.git/
```

Now that we have the entirety of the git repo locally, let's poke around at the contents, maybe the flag is in a previous revision.

After a long while of `git cat-file -p $objectHash`, I found a version of vampire.txt that shows us a specific time at which the flag will be available. Looking at our git log shows us that we don't have any commits at that time, it's in the future.

```
That time is Fri Jun 3 14:00:00 2016 +0530
```

# Solution
**Note:** I spent an incredibly long time on figuring this out, most of my ctf time went towards this problem. I probably overengineered it a lot. Below I show only the steps I took that actually led to solving.

Using the info we see in .git/config we are able to determine that this was hosted on github. Of course if we try to git clone that url it doesn't work, suggesting it may not exist anymore or it might have just been made private.

Knowing the author's git username now, let's do some snooping around on [his github](https://github.com/DhavalKapil). Looking at all his repositories we don't see much of interest. But hold on, he uses a gh-pages user page, which is hosted at [dhavalkapil.github.io]. Due to some recent research I was doing on github pages when trying to setup my own, I learned that a repo-page can be accessed by going to `user.github.io/repo-name`. Let's try that with what we already know and see if we see anything.

```
Flag is somewhere in this directory
```
That confirms that the git repo is still on github and it was just made private. Using the information we gleaned from the repository, let's put it all together now.

# TL;DR

1. CLUE has a .git folder that we can access specific files in
2. One of those specific files is `.git/config` which tells us where the remote/origin.
4. `Vampire.txt` tells us the flag will be in file `982hud0q3rhua` at some time
	* A previous version of `vampire.txt` tells us that the flag will be in that file at a specific time.
	* That specific time is in the future from where hack.bckdr.in/CLUE is currently at.
5. Github leaks us some information about private repos via gh-pages, provided we know enough information.
	* We need to know the repo name
	* We also need to know a specific file to probe
6. Visit the gh-page for the [repo](https://dhavalkapil.github.io/sdhf038efhsd0rw8eufsoid)
7. Visit the specific page that the password should be in on that gh-page

# Why it was vulnerable

It's important to note that this particular attack worked because vampire was doing his work on the repo in the gh-pages branch, so the repo was published and we just had to find the appropriate links. 

# Flag
This challenge is hosted permanently at [Backdoor](https://backdoor.sdslabs.co/challenges/CLUE), so go find the flag yourself!
