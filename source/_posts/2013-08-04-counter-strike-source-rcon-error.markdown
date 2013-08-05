---
layout: post
title: "Counter-Strike: Source RCON error"
date: 2013-08-04 20:22
comments: true
categories: css server
---

## Problem 

I was forced to reinstall my Counter-Strike server recently after the messy transition to SteamCMD and that action reintroduced a few minor issues along the way. One issue was with RCON, or Remote Console. This is a very easy fix but simple to forget about until you actually need it without having to restart the server.

I launch my server regularly with the following command in my mod folder (css):

```
$ ./srcds_run +map de_dust +maxplayers 16 -game cstrike
```

The port 27015 is forwarded on both TCP & UDP so I assumed I would not have any problems. However, if you try to use RCON, you may be presented with this glaring error:

```
] rcon meta list
Unable to connect to remote server (192.168.1.15:27015)
```

Seems odd since I'm obviously already connected to the server on that port to play the game.

## Fix

To fix this, simply add `+ip 192.168.1.xxx` to the launch command at startup.

Your command should now look like this:

```
$ ./srcds_run +map de_dust +maxplayers 16 -game cstrike +ip 192.168.1.15
```
