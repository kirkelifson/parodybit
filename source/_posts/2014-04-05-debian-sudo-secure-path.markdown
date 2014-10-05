---
layout: post
title: "Debian sudo secure\_path"
date: 2014-04-05 22:15:54 -0400
comments: true
categories: server
---

Debian has particular issues that irritate the hell out of me yet I hesitate to take even reasonable amounts of time to solve such problems, illogically. Tonight the feature/bug in question was a minor part of sudo compiled by default in Debian. I took learning Python/Flask upon myself suddenly because I felt I hadn't been programming enough as I should over the past few years. Misunderstandings about virtualenv, pip, and easy\_install aside I was asked in a guide to execute `sudo pip install python-virtualenv`, throwing me this error in return:

```
$ sudo pip install python-virtualenv
sudo: pip: command not found
```

Argh, fuck you, Debian! Executing `sudo env` showed me that `/usr/local/bin` was not a part of my PATH, and numerous attempts to use sudo flags got me absolutely no further. The man pages appeared to have let me down.

But alas,

According to [Debian Help](http://www.debianhelp.co.uk/sudo.htm) the problem lies within the --with-secure-path option that Debian maintainers think is vital to its reputation for being quite a secure and stable distribution (the 'unstable' branch is actually incredibly stable. Just saying.) Upon further inspection, the option adds a suggested path string to the top of the sudoers file, which can, and should be modified using `visudo`.

Prior to editing:
```
Defaults    secure_path="/sbin:/bin:/usr/sbin:/usr/bin"
```

Adding `:/usr/local/bin` to the end of the oh-so-special PATH for those pesky, untrustworthy sudoers (myself) solved the issue.
