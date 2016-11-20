---
layout: post
title: "RC3 CTF : Bridge of Cyber"
date: 2016-11-19 17:25:06 -0700
author: adam
slug: bridge-of-cyber
category:
- 2016
- rc3ctf
---
**Category:** Misc
**Points:** 200
**Description:**

>Welcome to the "Bridge of Cyber"! It's the same concept as the "Bridge of Death" in Holy Grail. Our DNS servers aren't very good at their job because they ask more questions then they answer. Let's see if you can get the flag from our DNS.

>Domain: misc200.ctf.rc3.club

# Investigation
I started by using the Dig tool to see what hints were available for this domain.

```
~$ dig ANY misc200.ctf.rc3.club
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;misc200.ctf.rc3.club.          IN      ANY

;; AUTHORITY SECTION:
ctf.rc3.club.           758     IN      SOA     ns-544.awsdns-04.net. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
~$
```

Looking at the results, the domain is hosted by the AWS Route 53 service which helps narrow down some of the options we can use. AXFR would have enabled us to download a copy of the entire domain set, but Route 53 does not current support it. In some deployments of DNSSEC, NSEC records are exposed which would enable subdomain enumeration exposing a list of subdomains for the entire domain, but again Route 53 does not support DNSSEC so we can't take advantage of those techniques.

At this point I got stuck because there did not appear to be any records that we could use to continue the challenge. Luckily they updated challenge description with a name server hint: ```Nameserver: ns-920.awsdns-51.net```.

Armed with new information, I switched to directly querying that name server for record information which proved much more fruitful:

```
~$ dig +nocomments ANY @ns-920.awsdns-51.net misc200.ctf.rc3.club
; <<>> DiG 9.9.5-3ubuntu0.10-Ubuntu <<>> +nocomments ANY @ns-920.awsdns-51.net misc200.ctf.rc3.club
; (2 servers found)
;; global options: +cmd
;misc200.ctf.rc3.club.          IN      ANY
misc200.ctf.rc3.club.   172800  IN      NS      ns-1526.awsdns-62.org.
misc200.ctf.rc3.club.   172800  IN      NS      ns-1842.awsdns-38.co.uk.
misc200.ctf.rc3.club.   172800  IN      NS      ns-489.awsdns-61.com.
misc200.ctf.rc3.club.   172800  IN      NS      ns-920.awsdns-51.net.
misc200.ctf.rc3.club.   900     IN      SOA     ns-1526.awsdns-62.org. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400
misc200.ctf.rc3.club.   300     IN      TXT     "What is the air-speed velocity of an unladen swallow"
;; Query time: 5 msec
```

Looking at the result set, we have NS records, a SOA, and a TXT. The NS records all pointed to Route 53, so I ignored those for now since Route 53. The SOA record also gave no more information since it was the default SOA record that Route 53 vends. The TXT record looked interesting since it had a Monty Python quote in it. Thanks to a [teammate](/about/dade), I knew that the next line in the quote was related to the African or European swallow.

```
~$ dig +short ANY @ns-920.awsdns-51.net european.misc200.ctf.rc3.club
"The roundest knight at king arthurs table was sir cumference. He acquired his size from too much __"
```

The next clue is a joke. Googling for the entire puzzle tells us that the punchline is "too much pi."

```
~$ dig +short ANY @ns-920.awsdns-51.net pi.european.misc200.ctf.rc3.club
"What is it that no man ever yet did see, which never was, but always is to be?"
```

The next hint is another puzzle, of which the answer is 'tomorrow'

```
~$ dig +short ANY @ns-920.awsdns-51.net tomorrow.pi.european.misc200.ctf.rc3.club
"My favorite things in life don't cost any money. It's really clear that the most precious resource we all have is ___"
```

This one is a quote from Steve Jobs and the answer is 'time'

```
~$ dig +short ANY @ns-920.awsdns-51.net time.tomorrow.pi.european.misc200.ctf.rc3.club
"RC3-2016-cyb3rxr05"
```

And Voilà, we have the flag.

# Flag
```
RC3-2016-cyb3rxr05
```
