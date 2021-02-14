---
title: "Update on Soluto and Antivir Solution Pro Virus"
date: 2010-08-04T22:26:00+08:00
excerpt: "In my previous post , I described my encounter with the Antivir Solution Pro virus after installing Soluto (\"anti-frustration software\" that analyzes the boot time of your PC). 
 Roee Adler, Chief Product Officer at Soluto, assured me the virus was not..."
draft: true
categories: [""]
tags: [""]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:  
>   
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/08/05/update-on-soluto-and-antivir-solution-pro-virus.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/08/05/update-on-soluto-and-antivir-solution-pro-virus.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx), I have copied it here in case that blog ever goes away.


In my [previous post](/blog/jjameson/archive/2010/08/01/soluto-and-antivir-solution-pro-virus.aspx), I described my encounter with the [Antivir Solution Pro virus](http://www.bing.com/news/search?q=antivir+solution+pro&amp;go=&amp;form=QBNT2) after installing [Soluto](http://www.soluto.com) ("anti-frustration software" that analyzes the boot time  of your PC).

Roee Adler, Chief Product Officer at Soluto, assured me the virus was not transmitted via Soluto (directly or indirectly via the PC Genome service). While they are aware of [some malware disguising itself as Soluto software](http://blog.soluto.com/2010/06/safety-notice-when-downloading-soluto-avoid-file-sharing-services/), this shouldn't be an issue provided you download the Soluto installer directly from [http://www.soluto.com](http://www.soluto.com) (as I did).

Note that I was unable to subsequently reproduce the virus infection on a clean Windows 7 VM using the same Soluto installer that I originally used on my Windows 7 desktop.

The very plausible explanation from the folks at Soluto is that my desktop was previously infected with the Antivir Solution Pro virus, but the virus was somehow hidden or "dormant" and consequently undetected by Microsoft Security Essentials (which itself is a very scary matter, but a different one altogether). When I installed Soluto, the virus was exposed and subsequently detected.

Note that in the new era of Software+Services, we are constantly having to decide which software and services we are willing to trust. In essence, each and every time you connect to the Internet -- either via a Web browser, or indirectly via some application that accesses services in the cloud -- you make a choice (perhaps subconsciously) that it is okay to trust the Web site or service and that nothing bad is going to happen as a result.

Given my experience with Soluto over the past several days, I firmly believe this is a company you can trust. You really should check out Soluto for yourself. While it didn't shave a lot of time off my PC boot process (down from 1:29 to 1:27), your "mileage" my vary significantly. If your PC takes several minutes to boot, Soluto can show you why -- and provide recommendations for reducing the boot time -- through a simple and intuitive UI.

Lastly, I wanted to include the email correspondence that I had with Roee. It's probably not all that interesting to most people, but it does demonstrate the level of pride these folks have in their product as well as the confidence they have in their development team.


> * * *
> 
> **From: **Roee Adler  
> **Sent: **Sunday, August 01, 2010 11:36 AM  
> **To: **Jeremy Jameson  
> **Subject: **MSDN Blogs: Contact request: Your Soluto Post
> 
> **Subject:** Your Soluto Post
> 
> Hi Jeremy, First - I'm sorry for the virus infection you went through, I'm sure it hasn't been fun. I'm 100%, totally confident it has nothing to do with Soluto. We actually heard of malware glued to Soluto installers that are spread around in torrents, but if you download Soluto from our homepage (or other trusted download sites), there's no way you got it from us. Where did you download it from? I'm sure from where you stand (seeing that you don't remember installing anything between your last test and installing Soluto) it looks like it's Soluto's fault but there's just no way... I myself am not a security expert but I can assure you once the team arrives in the morning they'll be on top of that to get some "proof". We're working in several channels with Microsoft, and we recently won TechCrunch Disrupt NYC, a highly prestigious start-up competition in NYC, I assure you we are a legitimate, honest start-up built of good-hearted coders. You can watch our interview where Tomer Dvir, Soluto's co-founder and CEO and I talk with Robert Scoble, an ex-Microsoft: [http://scobleizer.com/2010/05/24/why-can-soluto-do-what-microsoft-cant-they-get-rid-of-windows-frustrations-exclusive-first-look/](http://scobleizer.com/2010/05/24/why-can-soluto-do-what-microsoft-cant-they-get-rid-of-windows-frustrations-exclusive-first-look/) I invite you to respond and I'd love to have a phone conversation to further elaborate about Soluto. In the meanwhile, I urge you to recheck you assumption that Soluto caused the virus, because there are other explanations for this infection, and right now Soluto is accused on a public Microsoft blog of being a virus, and frankly I can't go to sleep until this error is fixed. To build more confidence I invite you to read what was written about us in LifeHacker, New York Times, DownloadSquad and many other publications that check their sources VERY well. Again, we are just a group of hard working software developers not hiding behind anything (you can see my name and face on our website), and we have nothing to do with your virus infection. I hope you'll find the reason for your infection and clear our name (and I know it's already seeded in Google that "Soluto" and "Virus" will appear together which is totally horrible, but at least the damage get be reduced by removing this post). Next time PLEASE contact before writing such things - hard-working people can get seriously hurt by such claims. And I apologize if anything I wrote in this note hurt you in any way, I'm quite in shock right now. All my contact details are below, I live in Israel, GMT+3. Best regards, Roee Adler Chief Product Officer, Soluto [...]
> 
> This contact request was sent from [MSDN Blogs](http://blogs.msdn.com/) by Roee Adler without sharing your email address.
> 
> * * *
> 
> **From: **Jeremy Jameson  
> **Sent: **Monday, August 02, 2010 4:04 PM  
> **To: **Roee Adler  
> **Subject: **RE: MSDN Blogs: Contact request: Your Soluto Post
> 
> Rose,
> 
> Thank you for contacting me. I look forward to working together to resolve this issue.
> 
> I am concerned about your statement about "malware glued to Soluto installers." However I believe that I downloaded the Soluto installer directly from [http://www.soluto.com](http://www.soluto.com). Note that I have no idea where the installer downloaded the additional setup files from. I assumed these also came from solute.com, but perhaps that is not necessarily the case.
> 
> I have attached the installer that I used yesterday to this message. It would be wonderful if you could have one of your engineers validate the checksum on this file against the "known good" installer.
> 
> In the meantime, I will install Soluto this morning on a clean Windows 7 VM to see if I can repro the issue. I will let you know the results ASAP.
> 
> Thanks,
> 
> Jeremy
> 
> * * *
> 
> **From: **Roee Adler  
> **Sent: **Monday, August 02, 2010 8:06 AM  
> **To: **Jeremy Jameson  
> **Subject: **RE: MSDN Blogs: Contact request: Your Soluto Post
> 
> Hi Jeremy.
> 
> Thanks for getting back to me. The file is proper (we compared the MD5) and signed by VeriSign, as well as our MSI.
> 
> If you downloaded the file from soluto.com, as I mentioned in my note to you, there's simply no way it has a virus. I really don't understand why you would assume the virus came with Soluto, after you've heard about us inside Microsoft, and why you choose to maintain the post that is so blatantly accusing us of being a virus, instead of at least taking it off in the meanwhile until you build confidence in us not being a virus. If your tests do find out we're malware -- I invite you to publish your findings, but right now I must say it's very insulting.
> 
> Your post was republished and retwitted several times by others, and our name is smeared in the meanwhile with no real justification. You can see how open we are about our problems in our support community, we hide nothing, and have an enormous amount of users. Why would you think we're a virus? Don't you think other users would have complained about such issues? By the way, our support community is powered by GetSatisfaction (as is clearly visible), a trustworthy company to which you need to authenticate in order to publish your questions/problems. We don't have access to your authentication information, they are kept by GetSatisfaction.
> 
> We've already heard from friends who encountered your post and were astonished by it. I hope you can take that into consideration and remove the blog post, it's hurts hard-working people every minute it's up.
> 
> Best,
> 
> Roee Adler  
> Soluto  
> [...]
> 
> * * *
> 
> **From: **Jeremy Jameson  
> **Sent: **Tuesday, August 03, 2010 1:35 AM  
> **To: **Roee Adler  
> **Subject: **RE: MSDN Blogs: Contact request: Your Soluto Post
> 
> Roee,
> 
> First of all, my apologies for misspelling your name on my initial email.
> 
> Second, I wanted to let you know that I did successfully install Soluto on a clean Windows 7 VM earlier today and I have not subsequently found any evidence of the Antivir Solution Pro virus on that VM.
> 
> At this point, I am as baffled as you are about how I got the Antivir Solution Pro virus yesterday morning. As I mentioned in my post, I did not browse any sites between the time I installed Soluto and the time I discovered the virus.
> 
> I also want to point out that I do **\*not\*** believe Soluto is malware. Rather, based on my experience yesterday and the sequence of events that transpired, it certainly **\*seemed\*** the virus was related to the install of Soluto or the subsequent communication with the PC Genome project.
> 
> Perhaps, as you mentioned, I somehow got the "malware glued to Soluto installers". However, since I downloaded the Soluto installer directly from [http://www.soluto.com](http://www.soluto.com) -- which you have also subsequently verified via MD5 checksum -- once again, I am simply at a loss for explaining (or, more accurately, speculating) what happened.
> 
> Is it possible the virus somehow came from another source? Absolutely.
> 
> If you are as confident as you can possibly be that the virus I encountered yesterday was in no way delivered via Soluto or the PC Genome service, then I'd like to post an update to my blog based on our correspondence.
> 
> As you pointed out in your earlier email, there are a number of sites that plagiarize blog posts from MSDN. Consequently, rather than removing my earlier post, I will post an update with the latest findings.
> 
> With your permission, I would like to include your email correspondence (without any contact info, of course) so that people are aware of what has transpired since my original post -- most importantly being the fact that I have been unable to reproduce the original issue (the infection with the Antivir Solution Pro virus).
> 
> As I stated in my original post, I really do look forward to recommending Soluto to my friends, family, and customers. I think it has great potential and I sincerely hope that nobody ever experiences anything similar to what I encountered yesterday.
> 
> Jeremy
> 
> * * *
> 
> **From: **Roee Adler  
> **Sent: **Tuesday, August 03, 2010 9:20 AM  
> **To: **Jeremy Jameson  
> **Subject: **RE: MSDN Blogs: Contact request: Your Soluto Post
> 
> Thanks Jeremy :)
> 
> One possible explanation can be that the installation of Soluto exposed the virus to the Security Essentials. I consulted several friends here who told me that these types of Trojans are sometimes capable of hiding themselves from whatever Anti-Virus solution is installed, so that Soluto's filter driver installation may have shifted loading orders in the kernel or something of that sort, that may have hurt the Trojan's ability to hide, just for enough time for the Security Essentials to catch it. This would mean that the Trojan infection was a pre-existing condition. Right now it's the best explanation I have. There's no way the virus got through the communication with the PC Genome. We use the highest levels of security, and a man-in-the-middle attack is not an option (taking into account that you're the only person to have had this problem with Soluto, out of our huge user-base). I don't know if you're aware of this, but the background of many of Soluto's team is in development of low level enterprise security applications, most notably Onigma, which was acquired by McAfee and now serves as McAfee's DLP solution.
> 
> About your update, you can also post my name and email if you wish, I invite everyone to contact me with questions, we try to be as open as possible.
> 
> When you say you'll post an update -- do you mean an addition to the same post, or a completely new one? If it's a new one, I'd appreciate it if you add to the old one an update clarifying the status.
> 
> Thanks again and please always feel free to contact me with questions regarding Soluto (or anything else).
> 
> Best,
> 
> Roee Adler  
> Soluto  
> [...]

