---
layout: post
title: "MITRE CTF 2014 - Forensics 300"
date: 2014-10-04 21:05:10 -0400
comments: true
categories: mitre ctf writeup
---

The VM we're given is a standard Ubuntu install from first glance and in order to access the system we're going to need to find credentials for the user georgesr. Rather than wasting time trying to guess or crack the password we can reboot the VM and hold down shift to open up the GRUB menu and boot into recovery mode. Inside the recovery menu we first enable networking in order to mount the system as read/write, and enter the root console.

Now with root access I started digging around the file system but didn't find interesting files. Much shame brought upon my family name when later on dej3cted asked, "Have you looked in the bash history?" Genius.

Contents of /home/georgesr/.bash\_history:
```
cd Documents/
zip -Pbluths contract-documents.zip contract-documents.txt
mcrypt contract-documents.zip
rm contract-documents.zip contract-documents.txt
```

So now we know the flag is most likely stored inside this archive and all we need to do is use the provided password to decrypt both and cash in on those sweet CTF points. Easy mode!

However, after opening the mcrypt container with the password that was used for the zip (just a guess) unzipping the file with the provided password does not work. That's odd. I tried copying the zip onto a remote machine in case anything funky was going on in the VM, but this did not work either.

At one point I resorted to brute-forcing the zip password using John the Ripper overnight and had the i7 in my MacBook ready to cook eggs for breakfast when I awoke. Nothing. Unfortunately we only had less than 24 hours left in the CTF so this option was not feasible. I also tried to use PKCrack to no avail.

During this time I looked to see if I could recover the original files from the trash but this also did not work (although the same files found in Documents were also still in the trash). Following these failures I started to look into the man pages for the zip/unzip commands. Was there something strange with these options, like spacing, that would cause such an issue? This did not help either.

A little bit later I reapproached the problem with a fresh perspective and started peeking into the provided binaries for zip/unzip. Were they directly modified or replaced?

```
# which zip
/usr/local/bin/zip

# which unzip
/usr/bin/unzip
```

Now that seems interesting. zip is not stored within the same directory as unzip, which suggests that a user installed a new binary to be used by default in replace of the original binary in /usr/bin.

```
# file /usr/local/bin/zip
/usr/local/bin/zip: Bourne-Again shell script, ASCII text executable
```

A shell script!

```
# cat /usr/local/bin/zip

#!/bin/bash
# Arguments = -P password

pass=
while getopts ":P:" opt; do
    case $opt in
        P)
            pass = $OPTARG"frozenbananas"
...
```

Ok now I think we've solved it. What the shell script does is call the original binary but instead of only using the password provided by the arguments it appends extra text as to foil our attempts to glean the information directly from the bash\_history. That was a pretty clever trick for a 300 point challenge.

```
# unzip -Pbluthsfrozenbananas contract-documents.zip
Archive:  contract-documents.zip
  inflating: contract-documents.txt
# cat contract-documents.txt
Looks like this is the last time you are going to ever see these...files.

-Kitty

P. S. 1670d90ab4a0cfa3dd429a7f203f801d47814f9b
```
