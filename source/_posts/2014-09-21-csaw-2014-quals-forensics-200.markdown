---
layout: post
title: "CSAW 2014 Quals - Forensics 200"
date: 2014-09-21 04:05:32 -0400
comments: true
categories: csaw ctf writeup
---

In this challenge we're given a packet capture and asked the simple question: 'why not sftp? well seriously, why not?'. My first instinct was to open the file in Wireshark and analyze the traffic, looking for anything that jumped out at me.

The challenge itself mentioned sftp the direct alternative to which is ftp. By looking at the data we see that there is indeed FTP traffic that has been captured. 

{% img https://i.imgur.com/DPLyA6R.png FTP Data %}

Following the TCP stream of the FTP traffic we can see through the logged data that the user opens a few directories then requests the file '/files/zip.zip'. That sounds promising.

Closing this TCP stream we come back to our protocol-sorted list of data and immediately underneath the FTP data is 'FTP-DATA' protocol traffic.

There are a few different TCP streams to look through (stream indices are displayed in the packet details frame). A quick glance at the length for the first two streams (90 and 131) as well as the hex tells me there probably isn't any valuable information contained inside.

The next stream contains quite a few more packets and after following the stream we are presented with the stream content window. I immediately spotted the text 'flag.png', as well as the 'PK' header (we are looking for an archive) so I knew this would be worth looking further into.

{% img https://i.imgur.com/wSvHN8T.png FTP-DATA stream %}

After clicking 'Save As', and saving the TCP stream to 'zip.zip', I `file` it to verify its type.

```
$ file zip.zip
zip.zip: Zip archive data, at least v2.0 to extract
```

Excellent, just what we were looking for. Extracting the files we find the file 'flag.png' contained inside and open it up. Success!

```
$ unzip zip.zip
Archive:  zip.zip
  inflating: flag.png
$ open flag.png
```

{% img https://i.imgur.com/9jXNXP9.png Flag! %}

Wireshark trickery:

A really fast way to do this is to use the filter "frame contains flag". The packet that contains the string 'flag' is immediately displayed and we can follow the TCP stream from there. This won't work 100% of the time for these kinds of challenges but it's worth trying to save time!
