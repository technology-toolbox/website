---
title: "The Original \"Jameson Datacenter\""
date: 2009-10-24T05:18:00-06:00
excerpt: "This morning I was doing some cleanup of my documents folder and I stumbled across a rather old Visio document that showed the beginnings of what I now refer to as the \"Jameson Datacenter\" (a.k.a. my home lab). For some geeky reason, seeing this again..."
aliases: ["/blog/jjameson/archive/2009/10/23/the-original-jameson-datacenter.aspx", "/blog/jjameson/archive/2009/10/24/the-original-jameson-datacenter.aspx"]
draft: true
categories: ["My System", "Personal", "Infrastructure"]
tags: ["My System", "Personal", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/24/the-original-jameson-datacenter.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/24/the-original-jameson-datacenter.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

This morning I was doing some cleanup of my documents folder and I stumbled
across a rather old Visio document that showed the beginnings of what I now
refer to as the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab). For some geeky reason, seeing this again brought a smile to my
face and a sense of nostalgia. It also caused me to recall two things:

1. That old Greatful Dead song "Truckin'" -- specifically the line
   
   <q class="directQuote">Lately it occurs to me: What a long, strange trip it's been.</q>
   
   > **Update**
   > 
   > Doh! You would think a guy with a Master of Science degree in Mechanical
   > Engineering could spell "Grateful Dead" correctly! I suppose that I really
   > should do more proofreading before clicking that **Publish** button ;-)

2. An article I read about a year ago describing the origin of Google that also included some photos of the original Google infrastructure in the Stanford University computer lab.

Now, please don't misunderstand -- I most certainly am not claiming that the
Jameson Datacenter will ever amount to even the most miniscule fraction of what
Google grew into in less than a decade. All I'm saying is that in the incredible
way the neurons are wired together in the human brain, things that we see very
often trigger distant memories with amazing clarity.

For those of you that may not have seen the original Google server farm, I did a
quick search this morning and managed to find the following:

{{< reference title="Google Stanford Hardware"
linkHref="http://geektechnique.org/media/google/googlehardware.html" >}}

If you haven't seen this, take a quick peek. It will surely bring a smile -- if
not a burst of laughter.

After enjoying the nostalgia for a moment, I almost deleted the Visio diagram
and continued my cleanup effort. However, then I decided that I might as well
"archive" this on my blog. It definitely won't help your efforts of deploying
Microsoft technology, but I think it's okay for a blog to wander a little every
now and then -- perhaps for no other reason than to keep you guessing as to what
may come next on the Random Musings of Jeremy Jameson.

Here's the physical architecture of what I now refer to as the "Old Jameson
Datacenter":

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Old-Jameson-Datacenter-System-Architecture-600x460.png"
alt="Old Jameson Datacenter - Physical Architecture" height="460" width="600"
title="Figure 1: Old Jameson Datacenter - Physical Architecture" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Old-Jameson-Datacenter-System-Architecture-987x757.png)

Apparently the last time I updated this Visio diagram, I still hadn't decided
what to do with all of the Dell servers I had procured through various channels
(primarily the Dell Outlet and ebay). Hence the Dell PowerEdge 4300 with the
"??" next to it. As I mentioned in my original post on the Jameson Datacenter,
this server was the original BEAST (which is now my "Production" database
server). However, it has since been replaced with a home-built server.

From the previous figure, you can start to see why I've consolidated these
servers in recent years by leveraging virtualization. The power consumption by
all of these servers was really rather ridiculous when you consider what I was
using these servers for.

Notice that I use to run a back-to-back firewall configuration with a perimeter
network (a.k.a. DMZ):

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Old-Jameson-Datacenter-TCP-IP-Configuration-600x471.png"
alt="Old Jameson Datacenter - TCP/IP Configuration" height="471" width="600"
title="Figure 2: Old Jameson Datacenter - TCP/IP Configuration" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Old-Jameson-Datacenter-TCP-IP-Configuration-960x754.png)

This is because I had one server with a public (static) IP address and therefore
ran a couple of Web sites on the old "ICEMAN" server -- hence the name "ICEMAN",
because it was my "IIS" server. Get it? Yeah, I know...more geekiness.

The naming convention was such that the names corresponding to
[X-Men](http://en.wikipedia.org/wiki/X-Men) heroes were all members of the
TECHTOOLBOX domain, whereas servers corresponding to X-Men villains were
considered "dangerous" and thus not members of the domain. Egad! Does the
geekness ever end?! [I wonder if someday when my daughter is much older, she'll
ever bother to read this and seriously start to question her father's sanity.]

Anyway, here's the DNS and DHCP configuration for the original farm:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Old-Jameson-Datacenter-DNS-Configuration-600x186.png"
alt="Old Jameson Datacenter - DNS/DHCP Configuration" height="186" width="600"
title="Figure 1: Old Jameson Datacenter - DNS/DHCP Configuration" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Old-Jameson-Datacenter-DNS-Configuration-794x246.png)

There's nothing really too interesting about that, I suppose.

Perhaps the most laughable aspect of the Old Jameson Datacenter -- at least when
compared to today -- was the storage infrastructure:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Old-Jameson-Datacenter-Disk-Configuration-600x475.png"
alt="Old Jameson Datacenter - Disk Configuration" height="475" width="600"
title="Figure 1: Old Jameson Datacenter - Disk Configuration" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Old-Jameson-Datacenter-Disk-Configuration-697x552.png)

I can still remember when I thought the 18 GB of RAID1 and 72 GB of RAID5 on
that Dell 4300 was
[fashizzle](http://www.urbandictionary.com/define.php?term=fashizzle) ;-) [Of
course, that was long before I was even aware of the term "fashizzle", but you
get my point.]

Anyway, I hope you enjoyed this brief diversion down Memory Lane. Perhaps it
invoked some distant memories of your own experiences in the world of
technology.

Personally, as I'm wrapping up this post, my head is spinning with thoughts of
my father's original Apple II+ (which was the first computer I ever "programmed"
on) as well as my very own Commodore 64 and all those late nights my dear friend
[Chris Gatewood](http://www.imediaconnection.com/profiles/iMedia_PC_Bio.aspx?ID=2928)
and I sat in front of it typing in hundreds of lines of code from a magazine
just so we could try to play some silly game based on "sprites." [Note that the
games almost never worked due to some error or another, but nevertheless we kept
trying anyway.]

