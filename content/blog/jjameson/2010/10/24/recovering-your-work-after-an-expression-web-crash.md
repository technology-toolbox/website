---
title: "Recovering Your Work After an Expression Web Crash"
date: 2010-10-24T04:02:00+08:00
excerpt: "I am getting a little tired of Expression Web 4 crashing on me. 
 I'm not sure why I'm repeatedly encountering issues with the latest version of Expression Web, but I suspect -- given the frequency at which it is crashing -- it may have something to..."
draft: true
categories: ["Development"]
tags: ["Debugging", "Web Development"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/10/24/recovering-your-work-after-an-expression-web-crash.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/10/24/recovering-your-work-after-an-expression-web-crash.aspx)
> 
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

I am getting a little tired of Expression Web 4 crashing on me.

I'm not sure why I'm repeatedly encountering issues with the latest version of  Expression Web, but I suspect -- given the frequency at which it is crashing --  it may have something to do with the TFS integration. Note that this is purely a  guess on my part, but I find it hard to believe that the memory corruption bug I'm  experiencing would not have been caught by one of the SDETs (a.k.a. Testers) on  the Expression Web team.

Perhaps the source of my woes is not the TFS integration at all, but rather something  to do with the fact that I use a non-trivial ASP.NET master page when creating/editing  pages. I guess it really doesn't matter -- I just want it to stop crashing regardless.

As I noted in a [previous post](/blog/jjameson/2009/09/12/expression-web-my-msdn-blog-and-now-team-foundation-server), I've been using Expression Web for a number of years to manage  content on my MSDN blog. While I'm obviously a little irritated this morning with  the tool, overall I'm still satisfied with my method of creating and editing HTML  content. It sure beats using the Web-based editing features in the Telligent Community  platform. [That's not meant to bash the Telligent functionality; for many people  -- heck, perhaps thousands and thousands of folks out there -- the "WYSIWYG" editors  provided by Telligent are probably more than sufficient to address their needs.  I simply prefer much tighter control over the HTML content...but, alas, I digress.]

[Ugh...the application just crashed again on me (second time this morning). Thankfully  I had just pressed <kbd>CTRL+S</kbd> about a minute ago -- so the "damage" wasn't nearly as  bad as it was earlier this morning.]

A few hours ago I was authoring a new blog post (not this one -- a different  one that hopefully more people will find valuable than this one) and after about  45 minutes of typing, revising, and typing some more, Expression Web suddenly crashed:

```
Problem signature:
  Problem Event Name:	BEX
  Application Name:	ExpressionWeb.exe
  Application Version:	4.0.1165.0
  Application Timestamp:	4bfaf4bc
  Fault Module Name:	StackHash_0a9e
  Fault Module Version:	0.0.0.0
  Fault Module Timestamp:	00000000
  Exception Offset:	00000000
  Exception Code:	c0000005
  Exception Data:	00000008
  OS Version:	6.1.7600.2.0.0.256.1
  Locale ID:	1033
  Additional Information 1:	0a9e
  Additional Information 2:	0a9e372d3b4ad19135b953a78882e789
  Additional Information 3:	0a9e
  Additional Information 4:	0a9e372d3b4ad19135b953a78882e789
```

At first, I was horrified. Almost an hour's worth of work down the drain!

In hindsight, I can't believe I didn't save my work-in-progress. [That "auto-save/auto-recover"  functionality in the various Microsoft Office apps (that everyone is now accustomed  to -- including me) *really* should be mandatory for all "desktop" applications  created by Microsoft.]

After resisting the temptation to curse something I won't type here -- or, even  worse, slam my fist into the keyboard (come on, we've all had those moments) --  I took a deep breath and decided to actually try to do something constructive for  a change. In the past couple of months, when Expression Web crashed, I would simply  click the link to send my crash info to Microsoft (a.k.a. the Watson bucket) and  restart the application. Then I would open my Web site again and start typing the  lost work as best I could from memory. [Sending crash reports to Microsoft is definitely *constructive* -- in that it helps identify problematic code -- but it certainly  doesn't address your immediate desire to recover your lost work.]

The very first thing I did was to take a screenshot of Expression Web so that  at least I'd have *some* of the content available verbatim to type over again.  Yes, this is admittedly very crude and almost certainly won't allow you to capture  everything -- especially if you tend to work in "Split" view (code + WYSIWYG) like  I do. However, it's still better than nothing.

The next thing I did was to choose the option to start debugging the application  (instead of sending the crash info to Microsoft and restarting). I chose to launch  a new instance of Visual Studio for debugging purposes.

Inside Visual Studio, I clicked the **Debug** menu and then clicked **Save Dump As...**

I specified the location and name for the mini-dump file and saved it.

I then closed Visual Studio and opened the dump file using WinDbg. Note that  you might be able to do what I'm about to cover using Visual Studio but, honestly,  I don't know how -- so instead I tend to use WinDbg for this kind of stuff.

Also note that you can also jump straight to WinDbg -- instead of launching Visual  Studio -- assuming you already have that installed. [I didn't at the time of the  crash since I rebuilt my desktop a few months ago and I haven't used WinDbg in a  long time. (99.999% of my debugging is done on custom code written by me -- or one  of my teammates -- through Visual Studio.)

You could also use something like ADPlus to create the mini-dump as well.

How you get the dump really doesn't matter...just be sure you get it.

I subsequently installed the [Debugging
Tools for Windows](http://www.microsoft.com/whdc/devtools/debugging/default.mspx), so that I'd once again have access to WinDbg. Note that the  install process has changed for this since the last time I used it. You now have  to download the debugging tools through the Windows SDK Setup Wizard. I recommend  you select the **Debugging Tools** option under **Redistributable
Packages**, and subsequently run **C:\Program Files\Microsoft SDKs\Windows\v7.1\Redist\Debugging
Tools for Windows\dbg\_x86.msi** to install the 32-bit version of the debugging  tools (since Expression Web 4 is a 32-bit app). Be aware that opening a dump file  from an x86 process in the x64 debugging tools will give you [a rather cryptic error message](http://www.bing.com/search?q=WinDbg+0n193).

Once you have the debugging tools installed, start WinDbg and open the dump file.

From the specific details related to my crash instance, I discovered that Expression  Web 4 has a mixture of managed and unmanaged code. Excellent! (For me at least,  since I'm working almost entirely in the "managed" world these days.)

Therefore, the first thing I did (after setting up my symbol path again -- **SRV\*C:\NotBackedUp\Symbols\*http://msdl.microsoft.com/download/symbols**)  was to load the SOS extensions used for debugging managed code:

```
.loadby sos clr
```

Hoping that my "lost" work (i.e. the HTML content) was stored somewhere in memory  as a managed string, I decided to go looking for all of the "large" strings in the  process at the time of the crash:

```
!dumpheap -strings -min 500
```

My assumption (or at least what I was hoping) was that the HTML I was looking  to recover was stored somewhere -- perhaps even "chunked" -- in a 500 byte string  (or strings -- plural). Unfortunately, after a quick scan of the 88 objects returned  from my dump file failed to produce anything that looked even remotely like my HTML  content, I had to go a little deeper.

I then scanned the entire memory process for a phrase (i.e. "Agilent solution")  that I knew existed in the content I had written:

```
s -u 0 L?0xffffffff "Agilent solution"
```

If you are not an expert with WinDbg -- don't worry, neither am I -- you just  need to know that this command is used to search the entire 32-bit memory address  space (i.e. 0x00000000 to 0xFFFFFFFF) for Unicode strings that contain the specified  characters.

In my case, this returned several hundred results. However, I got lucky (which  was quite nice in light of this morning's events) because I "hit the jackpot" with  the first address returned:

```
du /c 100 0f6dfa22
```

For those not familiar with WinDbg, this command is simply "dumping as Unicode"  the memory address (0x0f6dfa22), specifying a width of 100 columns (simply to make  the content easier to read).

Here's the output from this command:

0:000&gt; <kbd>du /c 100 0f6dfa22</kbd>

```
0f6dfa22  "Agilent solution did not involve the use of any OS or .	SharePoint language packs and thus required "custom" localization .	functionality, whereas the KPMG solution followed the more typical approach .	of installing language packs and leveraging "
0f6dfc22  "the "out-of-the-box" .	localization functionality.</p>.	<p>The "out-of-the-box" localization that I'm re"
```

Since this looked to be exactly what I was looking for, I opened the **Memory** window, typed in the address (0x0f6dfa22), and then clicked the **Previous** and **Next** button a few times to locate  the beginning and end of my content. Once I knew the "bounds" of my content, I then  dumped the entire content...

```
du /c 100 0f6de52a 0f6e0434
```

...and copied it to the clipboard. SHAZAM! No need to recall my content from  my memory (which is far from "[photographic](http://en.wikipedia.org/wiki/Eidetic_memory)").  Woohoo!

Yes, it's definitely "kludgey" -- and sure, I had to spend a few minutes tweaking  the pasted content to get it back to valid HTML -- but it worked (and not to mention  it made me a little proud to have recovered my work).

At this point, you might be asking something like "What if you didn't get lucky  with the first memory address? Does that mean you have to start manually inspecting  potentially hundreds of memory locations?"

The answer -- thankfully -- is "no." You can just loop through all of the memory  locations and dump each one:

```
.foreach(addr {s -[1]u 0 L?0xffffffff "Agilent solution"}){du /c 100 addr;.echo 
********}
```

This command assigns each memory address to a variable ("addr") and subsequently  runs the "dump Unicode" command on each address. It's going to generate a lot of  output, but hopefully you can find what you are looking for fairly quickly simply  by scanning the strings.

Lastly, I want to stress that this isn't meant to be a "tool" for frequent use.  Instead, I'm hoping at least of few people out there benefit from my experience  this morning (perhaps not even when using Expression Web, but some other application  entirely).

[I should also add that Expression Web crashed four times during the process  of writing this post. I think I'll reach out the Expression Web team and see if  I can get them to take a look at my mini-dump file. Hopefully they'll be able to  identify the root problem and develop a QFE (a.k.a. "hotfix"). I'll add an update  to this post if they do.]

Later today, I'll try to finish up [the blog post I originally wanted to publish today](/blog/jjameson/2010/10/25/localization-and-sharepoint-solutions-part-1).

