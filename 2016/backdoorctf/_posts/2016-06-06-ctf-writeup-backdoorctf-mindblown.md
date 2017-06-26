---
layout: post
title: "backdoorctf16 : Mindblown"
author: aagallag
date: 2016-06-06 13:49:00 -0700
category:
- 2016
- backdoorctf
slug: mindblown
---
**Category:** Crypto
**Points:** 150
**Description:**

>Chintu has secured his flag behind a secure auth. We managed to get the authentication logic of the system. Can you help us get his flag from here (http://hack.bckdr.in:9003/). Bruteforce is not needed.
<br />Created by: Amanpreet Singh

# Investigation
Upon visiting the site, I'm greeted with a simple login page.

<img src="{{ site.static }}/2016/backdoorctf/mindblown/1_login.png" class="img-responsive"/>

Right away I start trying random things in the login box to see how the page responds.  I try the combo 'admin:admin' and the webpage throws an error message stating "Username is wrong".

<img src="{{ site.static }}/2016/backdoorctf/mindblown/2_baduser.png" class="img-responsive"/>

Hmm...  Is this a typo or do I need to figure out the correct user to be using?

Thankfully, the challenge authors have provided a snippet of the login code.  Maybe I can better understand the challenge once I've reviewed some of the login implementation.

```js
var express = require('express');
var app = express();
var port = process.env.PORT || 9898;
var crypto = require('crypto');
var bodyParser = require('body-parser')
var salt = 'somestring';
var iteration = /// some number here;
var keylength = // some number here;

app.post('/login', function (req, res) {
	var username = req.body.username;
	var password = req.body.password;
	if (username !== 'chintu') {
		res.send('Username is wrong');
		return;
	}
	if (crypto.pbkdf2Sync(password, salt, iteration, keylength).toString() === hashOfPassword) {
		if (password === 'complexPasswordWhichContainsManyCharactersWithRandomSuffixeghjrjg') {
			// some logic here and return something
		} else {
			// return flag here
		}
	} else {
		res.send('Password is wrong');
	}
});
```

The following check tells me the username to be using.

```js
if (username !== 'chintu')
```

Great, now I know to use 'chintu' as my username.

<img src="{{ site.static }}/2016/backdoorctf/mindblown/3_badpass.png" class="img-responsive"/>

I try out the username, and the error changes from wrong user to wrong password.  How do we login as 'chintu'?

```js
if (crypto.pbkdf2Sync(password, salt, iteration, keylength).toString() === hashOfPassword)
```

So this is how I would expect most website logins to work.  It's bad practice to store passwords in plaintext, so you use a hashing algorithm to store the password hashes instead.  So when someone logs in, you calculate the hash of the password they claim is their's and check to ensure that it matches the password hash stored in a database.

However, in this implementation, there is one additional check performed even after the password hashes match.

```js
if (password === 'complexPasswordWhichContainsManyCharactersWithRandomSuffixeghjrjg') {
	// some logic here and return something
} else {
	// return flag here
}
```

Hmm, so Chintu's password appears to be 'complexPasswordWhichContainsManyCharactersWithRandomSuffixeghjrjg'.  To verify that suspicion, I went ahead and logged into the page with it.

<img src="{{ site.static }}/2016/backdoorctf/mindblown/4_badhash.png" class="img-responsive"/>

I was taken to a page different from the "Wrong username" and "Wrong password" pages.  However, the page still does not contain the flag.

Ok, so I know I'm using the right password and generating the right hash, but how do I get it to print the flag?

**We need to find a password that results in the same hash value as Chintu's password, but it can't be Chintu's password.**

How is that possible?  Thanks to **hash collisions**!

Hash functions have the interesting characteristic where you can input any large amount of data and they will always generate a hash of a fixed size.  This can be useful, because it means that hashes don't contain enough information to perfectly reverse a hash value to it's input.  The downside is that there often exists multiple variations of input that all can generate the same value.

# Solution
I was aware that collision attacks exist, but I've never implemented one.  So I decided to hit Google up for some help.  Specifically, I knew I would need help building a collision attack against pbkdf2Sync.

Searching "pbkdf2Sync collision" lead me to a writeup entitled [PBKDF2+HMAC hash collisions explained](https://mathiasbynens.be/notes/pbkdf2-hmac) by Mathias Bynens.  He does a fantastic job of showing exactly how to create a hash collision when you know the value being hashed.

# Flag
This challenge is hosted permanently at [Backdoor](https://backdoor.sdslabs.co/challenges/MINDBLOWN), so go find the flag yourself!
