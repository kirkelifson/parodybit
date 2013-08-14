---
layout: post
title: "Generating entropy for GPG"
date: 2013-08-14 20:24
comments: true
categories: gpg
---

When I tried generating a new GPG pair for my email yesterday I opted for a 4096 bit key, the highest possible choice. When doing so you'll most likely come across a message resembling the following:

```
Not enough random bytes available.  Please do some other work to give the OS a chance to collect more entropy!
```

Now, it gives you suggestions like generating hard drive or network activity which I tried doing by running `cat /dev/urandom > /dev/null`, `nmap -sV -O -PN 172.0.0.0/24`, and `tcpdump -vv > moredata` all to no avail.

I came across a solution that entails generating random numbers. To do this on Debian simply do the following:

```
$ sudo apt-get install rng-tools
$ rngd -r /dev/urandom
```

After starting that command, it should only take a few seconds for entropy to become sufficient for GPG.

