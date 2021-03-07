---
title: "My first bug fix for Subtext now available on GitHub (a.k.a. Building TechnologyToolbox.com, part 18)"
date: 2012-01-30T13:31:04-07:00
excerpt: "I finally got around to investigating all the hubbub with GitHub. I even submitted my first \"official\" bug fix for the Subtext blog engine. I'm sure it will get easier, but this experience was valuable for me, if for no other reason than making me appreciate how much I love Team Foundation Server."
aliases: ["/blog/jjameson/archive/2012/01/30/building-technologytoolbox-com-part-18.aspx"]
draft: true
categories: ["Development"]
tags: ["Subtext"]
---

In 	[one of last week's posts](/blog/jjameson/2012/01/23/building-technologytoolbox-com-part-15), I detailed a number of errors that I have encountered  	since switching my blog over to Subtext when I left Microsoft almost 4 months  	ago. In that post, I also described a few of the bug fixes that I made in the  	Subtext code in order to mitigate some errors.

Before any of those changes, however, I made a very simple change to Subtext  	2.5 in order to work out the processes for debugging and subsequent build/deployment.  	[Since the Technology Toolbox site is built using a Visual Studio 2010 solution  	and the Subtext 2.5 solution is built using a Visual Studio 2008 solution, this  	isn't quite as straightforward as you might think. It certainly isn't rocket  	science, but it's not quite as easy as pressing F5 either.]

That very first change I made to Subtext is actually for a legitimate bug  	(in other words, not an enhancement), but one that has probably gone undiscovered  	by most Subtext users and developers thus far. I didn't decide to fix the issue  	due to the "severity" of the bug, but rather based on the fact that the code  	change was about the simplest I can imagine.

If you look at the HTML for a Subtext blog page, you'll discover it specifies  	a DOCTYPE of **XHTML 1.0 Transitional** but actually contains invalid  	XHTML:

```
<link id="opensearch" rel="search" type="..." href="..." Title="..." />
```

See the problem? The **Title** attribute should be **title**.

I discovered this after copying the HTML for one of my Subtext-generated  	blog pages into Expression Web.

Like I said before...certainly not a very important bug, but still something  	I thought I should fix. As my nephew, Kyle, likes to say, "That's just how I  	roll."

So I fixed this bug in my own private build of Subtext way back in September,  	but that obviously doesn't help others who are using the Subtext blog engine.  	Thanks to 	[a recent post by Scott Hanselman](http://www.hanselman.com/blog/GetInvolvedInOpenSourceTodayHowToContributeAPatchToAGitHubHostedOpenSourceProjectLikeCode52.aspx), I decided to "bite the bullet" and share  	my changes via GitHub.

Okay, truth be told, I've been meaning to check out the Git version control  	system for quite a while now, but I kept putting it off since there are seemingly  	infinite things to learn about in the world of software development and a rather  	small number of free hours in the day. Don't get me wrong, I love Team Foundation  	Server and I'm definitely not looking to become a Git expert. However, I do  	like to at least be aware of various alternatives.

Consequently, last Friday I created an account on GitHub and then started  	going through some tutorials. It took a while to get setup -- partially because  	I somehow lost my original SSH key (I believe due to the fact that I use a roaming  	profile) -- but I eventually got to the point where I could checkout a file  	from GitHub, make a change, and subsequently check it back in. [Yeah, I know,  	I know...Git uses different terminology, but I'm a TFS guy -- what do you expect?]

Anyway, if you are interested in seeing my change, take a look at 	[my Subtext fork on GitHub](https://github.com/jeremy-jameson/Subtext).  	[Crap...I just noticed that 	[my commit from earlier today](https://github.com/jeremy-jameson/Subtext/commit/462934a87bd12649582f334545d3586b3c9f93a2) displays "unknown" even though I know that  	I set my user name and email address during the GitHub setup process. It looks  	like these got lost from my profile along with the SSH key. What a pain in the  	arse.]

I've also created a "pull request" for Phil to merge my fix into the "main"  	Subtext branch -- er, I mean his original repository for Subtext. [Again with  	the TFS terminology, I know.]

I'll work on applying the rest of my Subtext changes to my Subtext fork on  	GitHub in the near future.

