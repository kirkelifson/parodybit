---
layout: post
title: "Installing Postfix &amp; Dovecot on Debian Wheezy"
date: 2013-07-27 15:28
comments: true
categories: sysadmin postfix dovecot debian
published: false
---

While installing Postfix and Dovecot for the first time last August I noticed a lacking number of helpful guides to installing and maintaining the MTA (Mail Transfer Agent) and MDA (Mail Delivery Agent) cohesively into functional packaged backed by MySQL authentication. Whenever I did stumble upon an article that appeared promising it was terribly out-of-date and therefore very unhelpful. Recently Linode updated their [Library](https://library.linode.com/email/postfix/postfix2.9.6-dovecot2.0.19-mysql) with a great guide to get people started with services on an Ubuntu VPS.

It's a fantastic guide, and I recommend you read over it if you're interested in understanding how the packaged system manages mail on a conceptual basis. I will be using quite a bit of it as the basis for this tutorial. However, I prefer Debian and found that there were issues with the guide that resulted in the process not being distribution agnostic and failing to install properly.

This guide is solely for installation on a Debian Wheezy (7.0) installation. I assume that you already have everything else on the server installed and only wish to add mail functionality.


``` plain Current versions (07/24/13)
Postfix: 2.9.6
Dovecot: 2.2.4
```

First things first. Upgrade your package list to ensure everything is installed correctly through aptitude.

```
$ apt-get update && apt-get upgrade
```

Installing Postfix

Grab these packages listed

    ```
    $ sudo apt-get install postfix postfix-mysql postfix-ldap
    ```

    Now it's time to configure Postfix with MySQL. This article assumes you already have MySQL installed.

    ```
    $ mysql -u root -p
    mysql> create database mailserver;
    mysql> grant select on mailserver.* to mailuser@localhost identified by 'password!#123';
    mysql> flush privileges;
    ```

Postfix Virtual Domains

    mysql> CREATE TABLE `virtual_domains` (
      `id` int(11) NOT NULL auto_increment,
      `name` varchar(50) NOT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;

    testing postfix:
    postmap -q site.com mysql:/etc/postfix/mysql-virtual-mailbox-domains.cf
    postmap -q user@site.com mysql:/etc/postfix/mysql-virtual-mailbox-maps.cf
    postmap -q alias@site.com mysql:/etc/postfix/mysql-virtual-alias-maps.cf

Dovecot

The aptitude package for dovecot-common includes everything that you'll need.

```
$ sudo apt-get install dovecot-common
```
