---
title: "Soluto and Antivir Solution Pro Virus"
date: 2010-08-01T06:26:00-06:00
excerpt: "In one of the sessions I attended last week at TechReady (an internal training conference at Microsoft), the speaker mentioned a new piece of \"anti-frustration software\" called Soluto which analyzes the boot time  of your PC. It certainly sounded intriguing and I made a note to take a look at it when I got back home from Seattle.
This morning I  installed Soluto on my Windows 7 x64 desktop at home. Everything seemed great...at first...."
aliases: ["/blog/jjameson/archive/2010/07/31/soluto-and-antivir-solution-pro-virus.aspx", "/blog/jjameson/archive/2010/08/01/soluto-and-antivir-solution-pro-virus.aspx"]
draft: true
categories: ["Infrastructure"]
tags: ["Infrastructure", "Windows 7"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/08/01/soluto-and-antivir-solution-pro-virus.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/08/01/soluto-and-antivir-solution-pro-virus.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

> **Update (2010-08-05)**
>
> Note that I was unable to reproduce the virus infection after installing Soluto
> on a different environment. I encourage you to read [my next post](/blog/jjameson/2010/08/05/update-on-soluto-and-antivir-solution-pro-virus) instead of -- or in addition to -- this post.

In one of the sessions I attended last week at TechReady (an internal training conference         at Microsoft), the speaker mentioned a new piece of "anti-frustration software"         called [Soluto](http://www.soluto.com) which analyzes the boot time of         your PC. It certainly sounded intriguing and I made a note to take a look at it         when I got back home from Seattle.

This morning I installed Soluto on my Windows 7 x64 desktop at home. Everything         seemed great...at first.

Soluto reported that my boot time was 1 minute 27 seconds (showing how that timeline         was broken down loading various applications). It also recommended that I disable         the Microsoft Office Groove client as well as Adobe Acrobat Reader (both of which         seemed reasonable given that I never use Groove on this particular machine and I         rarely view PDF documents). Soluto also discovered some "unrecognized" programs         and prompted me for permission to connect to the PC Genome project to attempt to         identity them.

Unfortunately, after a few minutes I discovered that my PC was infected with the         [Antivir Solution Pro virus](http://www.bing.com/news/search?q=antivir+solution+pro&go=&form=QBNT2). This is a particularly nasty virus because it         disguises itself as an anti-virus program, disables other security measures, and         subsequently attempts to gather personal information. For example, when I attempted         to launch Microsoft Security Essentials, I received a message stating that the program         was infected. The virus also set the proxy on Internet Explorer to 127.0.0.1:5643         (which redirected all HTTP requests through the virus, undoubtedly in an attempt         to steal personal information).

I managed to avoid the land mines with the virus and quickly removed Soluto (which         also removed Antivir Solution Pro). I was then able to start Microsoft Security         Essentials, at which point it detected a Trojan horse on my computer (as shown in         the screenshot below).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Microsoft-Security-Essentials-trojans-600x420.png"
alt="Microsoft Security Essentials - trojans"
class="screenshot"
height="420"
width="600"
title="Figure 1: Microsoft Security Essentials - trojans" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Microsoft-Security-Essentials-trojans-800x560.png)

I am now running a "Full scan" with Microsoft Security Essentials just to be safe.

I'm not sure how Soluto managed to infect my computer, but I can tell you that I         did not surf any Web sites between the time I installed Soluto and the time I discovered         the Antivir Solution Pro virus.

Beware of Soluto. I love the idea, but now I'm very wary of the actual implementation.         It will be a long, long time before I attempt to install it again.

If you work for Soluto and you are reading this, note that I tried submitting a         post to [http://community.soluto.com](http://community.soluto.com), but         unfortunately your site requires me to authenticate before submitting to the forums.         I'm sure you can understand why I didn't feel comfortable registering any personal         information with your site in light of my experience today. However, please feel         free to contact me through my blog. I really would like to be able to eventually         recommend Soluto to my friends, family, and customers.

