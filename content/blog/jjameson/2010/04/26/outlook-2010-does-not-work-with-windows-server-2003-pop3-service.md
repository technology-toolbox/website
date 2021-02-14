---
title: "Outlook 2010 Does Not Work with Windows Server 2003 POP3 Service"
date: 2010-04-26T00:55:00+08:00
excerpt: "I've mentioned in the past how I run a Windows Server 2003 mail server in order to use the POP3 service for basic e-mail functionality, and that I didn't have any interest in finding an alternative when I discovered POP3 is no longer available in Windows..."
draft: true
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/26/outlook-2010-does-not-work-with-windows-server-2003-pop3-service.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/26/outlook-2010-does-not-work-with-windows-server-2003-pop3-service.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

I've mentioned [in the past](/blog/jjameson/2009/09/14/the-jameson-datacenter) how I run a Windows Server 2003 mail server in order to use the POP3 service for basic e-mail functionality, and that I didn't have any interest in finding an alternative when I discovered POP3 is no longer available in Windows Server 2008. Well, I guess I need to start investing the effort in finding a new email service to run for demo purposes after all.

After installing Office 2010 recently on my home desktop, I discovered that Outlook 2010 apparently considers the simple implementation of the POP3 protocol in Windows Server 2003 to be obsolete, because after upgrading from Outlook 2007, I can no longer connect to my mail server (BANSHEE) and receive email messages.

At first, I suspected the problem might somehow be caused by installing the new version of the Microsoft Outlook Hotmail Connector (which, for Outlook 2010, is currently still a beta version), but I verified that the problem still exists even after uninstalling it.

I also tried disabling the **Require Secure Password Authentication (SPA) for all client connections** option for the POP3 service, but I couldn't get it to connect even when sending my username and password in clear text over the wire (which obviously isn't a good idea anyway, so I suppose I should be glad that option didn't work).

Here's a network trace (when SPA is enabled) from my primary desktop (WOLVERINE) with Outlook 2010 installed:

```
449   3.055000   BANSHEE    WOLVERINE  POP3   POP3:Response: +OK: Microsoft Windows POP3 Service Version 1.0 <846424703@banshee.corp.technologytoolbox.com> ready.
450   3.055000   WOLVERINE  BANSHEE    POP3   POP3:Command: CAPA 
451   3.056000   BANSHEE    WOLVERINE  POP3   POP3:Response: -ERR, Error: Unacceptable command
452   3.056000   WOLVERINE  BANSHEE    POP3   POP3:Command: QUIT
```

Like I said, it seems that Outlook 2010 doesn't like the simple implementation of the POP3 protocol in Windows Server 2003. Specifically, if the POP3 server doesn't understand the CAPA command (to list the capabilities supported by the mail server), then Outlook 2010 doesn't even bother trying to authenticate (at least not with SPA).

Here is a similar network trace from one of my VMs (FOOBAR2) that still has Outlook 2007 installed:

```
2225   15.118164   BANSHEE   FOOBAR2   POP3   POP3:Response: +OK: Microsoft Windows POP3 Service Version 1.0 <846956359@banshee.corp.technologytoolbox.com> ready.
2229   15.210937   FOOBAR2   BANSHEE   POP3   POP3:Command: AUTH  
2230   15.212890   BANSHEE   FOOBAR2   POP3   POP3:Response: +OK
2231   15.241211   FOOBAR2   BANSHEE   POP3   POP3:Command: AUTH  NTLM
2232   15.243164   BANSHEE   FOOBAR2   POP3   POP3:Response: +OK
2234   15.243164   FOOBAR2   BANSHEE   NLMP   NLMP:NTLM NEGOTIATE MESSAGE
2235   15.245117   BANSHEE   FOOBAR2   NLMP   NLMP:NTLM CHALLENGE MESSAGE
2236   15.245117   FOOBAR2   BANSHEE   NLMP   NLMP:NTLM AUTHENTICATE MESSAGE, Domain: TECHTOOLBOX, User: jjameson, Workstation: FOOBAR2
2239   15.335937   BANSHEE   FOOBAR2   POP3   POP3:Response: +OK: User successfully logged on
2240   15.336914   FOOBAR2   BANSHEE   POP3   POP3:Command: STAT 
2241   15.336914   BANSHEE   FOOBAR2   POP3   POP3:Response: +OK: 1 827
2242   15.337890   FOOBAR2   BANSHEE   POP3   POP3:Command: UIDL 
2243   15.337890   BANSHEE   FOOBAR2   POP3   POP3:Response: +OK: 1 messages (827 octets)
2244   15.337890   FOOBAR2   BANSHEE   POP3   POP3:Command: LIST 
2245   15.338867   BANSHEE   FOOBAR2   POP3   POP3:Response: +OK: 1 messages (827 octets)
2262   15.383789   FOOBAR2   BANSHEE   POP3   POP3:Command: RETR  1
2263   15.384765   BANSHEE   FOOBAR2   POP3   POP3:Response: +OK: 827 octects
2374   15.774414   FOOBAR2   BANSHEE   POP3   POP3:Command: DELE  1
2375   15.775390   BANSHEE   FOOBAR2   POP3   POP3:Response: +OK: Message marked as deleted
2433   15.819336   FOOBAR2   BANSHEE   POP3   POP3:Command: QUIT
```

I suspect the problem with Outlook 2010 might be due to the fact that I'm not using SSL to connect to my POP3 service. However, unless I'm missing something obvious, I don't believe it's even possible to configure SSL on the POP3 service in Windows Server 2003. Sure, I could change the port number from the default 110 to 995, but how would I assign the certficate?

I suppose this means I'll have to start looking at third-party email servers for demo purposes and learning about various mail-enabled features in other products and technologies, such as SharePoint. I really need something much more "lightweight" than Exchange to address my specific scenarios.

The POP3 service in Windows Server 2003 was great for demo and training purposes -- while it lasted. I suppose another option is to simply use a different e-mail client instead of Outlook 2010, but I'm sure you can imagine why that option doesn't sound very appealing.

> **Update (2010-04-27)**
>
> I started investigating alternate e-mail services this morning, but quickly discovered [there are a lot of potential candidates](http://www.emailman.com/win/servers.html) out there. [One of the simplest options](http://weblogs.asp.net/hpreishuber/archive/2008/04/30/visendo-smtp-pop3-extender-for-windows-2008-server.aspx) looked promising at first, but after reading through the multitude of comments describing various issues with it, I discovered that it doesn't appear to work with Outlook 2010 either.
>
> This morning, I decided to just "bite the bullet" and download Mozilla Thunderbird to see if I could use this to connect to my Windows Server 2003 POP3 service for demo and training purposes. It took less than 10 minutes to download, install, and configure my e-mail account. Note that Thunderbird automatically detected the DNS names of my POP3 and SMTP servers (which are really one and the same) just by typing in my e-mail address. The only "tricky" part was ensuring that I checked the option to **Use secure authentication** (since my POP3 service is configured to require SPA).
>
> I really don't like the idea of not using Outlook for demos, but unless someone can point me to a POP3 service that is free, lightweight (i.e. can run adequately in a VM with a mere 256MB of memory), and trivial to install and configure, then I'm going to stick with the Windows Server 2003 POP3 service for now.

