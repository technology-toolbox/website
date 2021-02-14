---
title: "Git/GitHub issues"
date: 2012-01-30T23:35:45+08:00
excerpt: "Follow along as I share my struggles using GitHub and the Git version control system."
draft: true
categories: ["Development"]
tags: ["Core 
			Development", "Subtext"]
---

In [last night's post](/blog/jjameson/2012/01/30/building-technologytoolbox-com-part-18), I mentioned how I recently created an account on GitHub (primarily so I can contribute the various bug fixes and enhancements that I've made to the Subtext blog engine).

I also mentioned a couple of issues that I encountered with Git -- specifically losing my SSH key (after logging out and subsequently logging back in) and the fact that [the first commit in my Subtext fork](https://github.com/jeremy-jameson/Subtext/commit/462934a87bd12649582f334545d3586b3c9f93a2) shows "unknown" (instead of my name).

I'm speculating that both of these issues are somehow related to the fact that my Windows account uses a roaming profile (and, for whatever reason, the changes were not saved to the server upon logout). On rare occasion, I've seen similar issues in the past (most recently, back in 2009 while working on the KPMG project).

This morning, when I logged back in to the development VM where I installed Git, I once again encountered the dreaded "Permission
denied (publickey)" error message. Argh!

After nuking my key files, generating a new key, copying the new public key into my GitHub account settings, and then setting the various configuration variables once again (e.g. <var>user.name</var> and <var>user.email</var>), I verified that I could once again successfully "<kbd>ssh -T git@github.com</kbd>".

Unfortunately, when I started Visual Studio and opened the Subtext solution, I noticed that none of the items in Solution Explorer showed any source control icons (à la the [Git Source Control Provider](http://visualstudiogallery.msdn.microsoft.com/63a7e40d-4d71-4fbb-a23b-d262124b8f4c) that I installed last week). I tried making a change to a file and, sure enough, Visual Studio was completely ignorant to the Git version control system (in other words, it treated the file as if it were simply flagged as read-only).

Suspecting this might be caused by the "SSH key reset dance" I performed once again earlier this morning, I nuked the Subtext folder and cloned the repository from GitHub. That at least got the source control functionality working once again in Visual Studio.

At this point, I logged off of this development VM and then logged back in -- to ensure my Git setup would not get lost again. Thankfully, it now seems to be working as expected.

However, since I'm apparently in a ranting mood this morning, I thought I should share something else. What is wrong with the following screenshot?

![Git - pending changes - yes or no?](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Git%20-%20Pending%20changes%20-%20yes%20or%20no.png)

    	Figure 1: Git - pending changes - yes or no?

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Git%20-%20Pending%20changes%20-%20yes%20or%20no.png)

So, which is it?

Are there pending changes, as indicated by the source control icons next to the **Subtext.Framework** and **Subtext.Web** projects? Or, are there no pending changes, as indicated by the empty list in the **Git Pending Changes** window? [...and yes, I did try clicking the **Refresh **button -- in case you think that might be the issue.]

The correct answer seems to be "this is a bug in the Git Source Control Provider for Visual Studio." I say this based on the output from "<kbd>git status</kbd>":

```
$ cd /c/NotBackedUp/Subtext/

jjameson@FOOBAR7 /c/NotBackedUp/Subtext (master)
$ git status
# On branch master
nothing to commit (working directory clean)
```

Well, as I've said repeatedly before, software is never perfect. My experience over the last couple of days and this morning also reminds me of another common saying: "You get what you pay for."

