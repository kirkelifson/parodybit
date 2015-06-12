---
layout: post
title: "Server configuration for autoconfig in Thunderbird and Outlook"
date: 2014-08-22 21:49:25 -0400
comments: true
categories: server
---

When I first looked at how to do this it appeared to be an enormous mess. The sight of XML made me want to vomit, and the vague concept of adding a subdomain that points to a specific web directory containing said XML made me want to vomit once more. But trust me, it's all simple and is rather intuitive! (Dear Mozilla and Microsoft, XML still needs to go.)

##Thunderbird

The real Thunderbird docs for this guide are located [here](https://wiki.mozilla.org/Thunderbird:Autoconfiguration). However it's a little long to read and not really worth reading anyways. You can save yourself the effort because you'll need it to respond to all of your emails!

1. Add a subdomain called `autoconfig.yourdomain.com` to your DNS records and point it to your web server.
2. Set up a nginx/Apache virtual host to serve one file. This file needs to be available at the URL `http://autoconfig.yourdomain.com/mail/config-v1.1.xml`

nginx:

```
server {
    server_name autoconfig.yourdomain.com;
    root /var/www/autoconfig/;

    listen 80;

    index config-v1.1.xml;
}
```

3. `$ mkdir -p /var/www/autoconfig/mail`
4. `$ vim mail/config-v1.1.xml`

```
<clientConfig version="1.1">
  <emailProvider id="yourdomain.com">
    <domain>yourdomain.com</domain>
    <displayName>Your Server Mail</displayName>
    <displayShortName>Your Server</displayShortName>
    <incomingServer type="imap">
      <hostname>yourdomain.com</hostname>
      <port>993</port>
      <socketType>SSL</socketType>
      <authentication>password-cleartext</authentication>
      <username>%EMAILLOCALPART%</username>
    </incomingServer>
    <outgoingServer type="smtp">
      <hostname>yourdomain.com</hostname>
      <port>465</port>
      <socketType>SSL</socketType>
      <authentication>password-cleartext</authentication>
      <username>%EMAILLOCALPART%</username>
    </outgoingServer>
  </emailProvider>
</clientConfig>
```

You may notice the part containing `%EMAILLOCALPART%`. My server is configured to only accept usernames that do not contain the domain name (e.g., kirk vs kirk@thisdomain.net). You can look [here](https://wiki.mozilla.org/Thunderbird:Autoconfiguration:ConfigFileFormat) for a full list of variables that can replace this.

All other server specific settings may need to be changed as well. It should be straight forward from here.

##Outlook

I actually haven't tested this yet because I don't own Outlook, but let me know if this does not work. I will be testing it myself sometime soon.

The setup is practically the same. However, the URL path and XML is different (what is standards?). The Microsoft doc for this guide is located [here](http://technet.microsoft.com/en-us/library/cc511507%28v=office.14%29.aspx). However, if you hate Microsoft documentation as much as I do just read below.

1. The URL you will need to provide for Outlook to detect your settings is: `https://yourdomain.com/autodiscover/autodiscover.xml` I know that this specifies SSL and Thunderbird's settings did not. I will look into whether or not it's required later. The nginx settings I used for this are:

```
server {
    server_name yourdomain.com www.yourdomain.com;

    location /autodiscover {
        alias /var/www/autoconfig/mail/;
        autoindex off;
    }
}
```

2. The contents of the XML file are as follows:

```
<?xml version="1.0" encoding="utf-8" ?>
<Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/responseschema/2006">
<Response xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a">
<Account>
    <AccountType>email</AccountType>
    <Action>settings</Action>
    <Protocol>
        <Type>IMAP</Type>
        <Server>yourdomain.com</Server>
        <Port>993</Port>
        <DomainRequired>off</DomainRequired>
        <SPA>off</SPA>
        <SSL>on</SSL>
        <AuthRequired>on</AuthRequired>
    </Protocol>
    <Protocol>
        <Type>SMTP</Type>
        <Server>yourdomain.com</Server>
        <Port>465</Port>
        <DomainRequired>off</DomainRequired>
        <SPA>off</SPA>
        <SSL>on</SSL>
        <AuthRequired>on</AuthRequired>
        <UsePOPAuth>on</UsePOPAuth>
        <SMTPLast>on</SMTPLast>
    </Protocol>
</Account>
</Response>
</Autodiscover>
```

Same as before: you will need to change the settings provided if your server is any different than mine. It should be obvious from the XML elements what needs to be changed.

That's it! Enjoy the automatic configurations (no more manual smtp and imap configs!!!). Also, stop using Outlook.
