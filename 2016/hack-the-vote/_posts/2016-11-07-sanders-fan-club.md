---
layout: post
title: "Hack The Vote 2016 : Sander's Fan Club"
author: dade
date: 2016-11-07 13:37:00 -0700
slug: sanders
category:
- 2016
- hack-the-vote
---
**Category:** Web
**Points:** 100
**Description:**

>Those deplorable Sanders supporters are still fighting. Shut the site down by finding where the idiot stored his credentials.<br />http://sandersfanclub.pwn.democrat

# Investigation
Visiting the site, we're presented with a pretty basic bootstrap site praising Bernie Sanders. Reading the content in the jumbotron suggets that the credentials were left laying around somewhere, and that it only works in Firefox. I was stubborn and thought this might be a red herring so continued to work in Chrome.

I fired up Fiddler and noticed that on every request, flag2.jpg is being sent in the HTTP `Link:` header and that it's set to `rel=stylesheet`. Unfortunately, however, I don't see any CSS in the body of what Fiddler sees. Okay, let's see what Firefox has to offer.

Visiting the page in Firefox, I can see that there are very clearly different styles being applied, so I open up the firefox inspector (right click -> Inspect Element) and start looking at the styles. Unfortunately I don't see anything other than rules from `bootstrap.min.css:5` so let's head to the network tab and see what happens when I refresh.

I notice right away that there are two GET requests for `flag2.jpg` but one is only 2.18KB. Looking at the headers between these two requests, it seems that in the first request we have `Accept: "text/css,*/*;q=0.1"` whereas in the second request we only have `Accept: "*/*"`. If we take a look at the response, we'll in fact see that the first request has valid CSS rules whereas the second request doesn't. At the bottom of these CSS rules we see:

```
/*
 * How did I... Nevermind. I'm pretty sure my creds are in a text file
*/
```


# Solution
So when we tell the web server that we want `flag2.jpg` and that we except it to be `text/css`, it seems to behave differently than if we don't expect it to be `text/css`.  And we know that the creds are in a text file, so let's tweak the Accept header and see what happens. This can be achieved easily by right clicking on the `flag2.jpg` request and choosing "Edit and resend".

```
GET http://sandersfanclub.pwn.democrat/flag2.jpg HTTP/1.1
Host: sandersfanclub.pwn.democrat
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:47.0) Gecko/20100101 Firefox/47.0
Accept: text/plain,*/*;q=0.1
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://sandersfanclub.pwn.democrat/
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
```

# Why it was vulnerable
I'm leaving this section here, as if there is some lesson to be learned, but to be honest I've never seen this before and I highly doubt it would ever be found in the wild. In reality, it seems like it's just abusing the RFC.

# Flag
```
Password reminder: flag{I_am_very_bad_with_computers}
(Go tell chrome devs to support RFC 5988 
Firefox masterrace)
```
