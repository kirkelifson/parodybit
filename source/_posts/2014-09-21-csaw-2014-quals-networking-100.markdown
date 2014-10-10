---
layout: post
title: "CSAW 2014 Quals - Networking 100"
date: 2014-09-21 04:06:12 -0400
comments: true
categories: csaw ctf writeup
---

This challenge was called 'Big Data' and apparently is 'Something, something, data, something, something, big'. Downloading the pcap tells us it actually is big, a full 26.7MB. Not as big as Big Data gets, but much larger than just a little bit of text.

I started to look through the packet capture and saw a ton of random packets that didn't point to anything specific. With 26,365 total packets in the capture scrolling might take quite a long time! I sorted by protocol and clicked at abitrary points on the scrollbar to see if anything good came up (I know now a few useful Wireshark and will explain below).

I took a look at a giant piece of UDP data and saw that it was BitTorrent traffic! I followed this lead for a few minutes, trying to figure out if it was possible to find the original torrent file location, magnet URL, tracker information, or even the data itself. I didn't find any promising leads and tried looking for anything else that could be useful in the packet capture.

By random chance I came across telnet data. Although the topic of the challenge was 'Big Data', it was the most interesting thing I had found in the data after looking through ARP requests and other bits of useless junk.

Following the stream immediately yields the flag (it's only 100 points for a reason).

{% img https://i.imgur.com/d31htbR.png Flag! %}

Useful Wireshark features:

Filters are very useful when trying to parse a large amount of information (one might call it Big Data). Using a filter such as "!tcp" or "!udp" could significantly shorten the amount of data to scroll through.

After looking for a filter that could yield the flag directly I found none. By inspecting the telnet data you can see that each character of the flag is sent in an individual packet so a regular search for the string "flag" would not work. I am not sure if there is a way around this as to search entire streams for data.
