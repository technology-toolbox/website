---
title: "Disk Benchmarks: SSD vs. \"Raptor\" vs. RAID"
date: 2011-03-11T08:00:00-07:00
excerpt:
  Earlier this week, I posted about how I rebuilt my Windows 7 desktop while
  installing a new solid-state drive (SSD). This morning, I thought I would
  share some performance numbers that I gathered from the various disk
  configurations that I currently...
aliases:
  [
    "/blog/jjameson/archive/2011/03/10/disk-benchmarks-ssd-vs-quot-raptor-quot-vs-raid.aspx",
    "/blog/jjameson/archive/2011/03/11/disk-benchmarks-ssd-vs-quot-raptor-quot-vs-raid.aspx",
  ]
categories: ["My System", "Infrastructure"]
tags: ["My System", "Infrastructure"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/03/11/disk-benchmarks-ssd-vs-quot-raptor-quot-vs-raid.aspx"
---

Earlier this week, I posted about
[how I rebuilt my Windows 7 desktop](/blog/jjameson/2011/03/09/windows-7-sp1-ssd-rebuild-and-maxpatchcachesize-0)
while installing a new solid-state drive (SSD).

This morning, I thought I would share some performance numbers that I gathered
from the various disk configurations that I currently have running in the
"[Jameson Datacenter](/blog/jjameson/2009/09/14/the-jameson-datacenter)" (a.k.a.
my home lab).

Note that prior to installing the SSD in my desktop, I previously used a Western
Digital 74 GB "VelociRaptor" drive as the primary disk in my desktop. In case
you are not familiar with the "Raptor" drives, the key characteristic is the
increased speed of the drive -- specifically 10,000 RPM. [A 74 GB hard drive is
obviously rather puny these days, but keep in mind that I paid a premium for
this drive ($189) back in 2005.]

A few years later, I swapped out some drives in one of my servers and added a
couple of 250 GB drives to my desktop in a RAID 0 configuration. This yielded an
additonal 500 GB of storage, which allowed me to replicate lots of content
previously stored only on one of my servers and my Media Center in the family
room (i.e. music, pictures, and video).

After adding the SSD this week, I now have three logical drives in my desktop
(four physical drives).

To summarize, here's the current storage configuration for my desktop
(WOLVERINE):

- C: - 119 GB (formatted) - Crucial RealSSD 128GB (C300-CTF DDAC128MAG)
- D: - 69 GB (formatted) - WD Raptor WD74 0GB-00FLC0 74GB
- E: - 465 GB (formatted) - RAID 0 - ST3250410AS 250GB (x2)

Here's a pivot chart that I put together that shows the performance
characteristics of each of these drives.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Benchmarks-WOLVERINE-600x404.png"
alt="Disk performance on WOLVERINE" height="404" width="600"
title="Figure 1: Disk performance on WOLVERINE" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Benchmarks-WOLVERINE-823x554.png)

The chart above clearly illustrates why performance is such as relative term
when talking about hardware. While the speed of the Raptor drive was quite
impressive back when I bought it, it's almost laughable now when compared to
what you can spend approximately $200 on these days (in terms of storage).

What I find most interesting about the above chart is that when it comes to
write performance, a RAID 0 array comprised of some relatively old disks is
actually faster than a brand new SSD. It's widely known that SSDs are much
faster with read operations, than write operations -- but until I put together
the above chart, I didn't truly understand the performance compared to other
alternatives.

Even though the SSD is signficantly faster than my RAID 0 configuration with
respect to reads (and almost as fast with respect to writes), it's not what I
would recommend in most scenarios, primarily because of the high $/GB. [FYI, if
I were to copy my music, pictures, and videos to my new SSD drive -- along with
everything else I've already installed on the C: drive -- I would come up short
by about 20 GB.]

To understand why I wouldn't recommend an SSD for most scenarios, take a look at
the following chart which shows the performance of the SSD on my desktop
(WOLVERINE) compared with other hard drives running on various servers.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Benchmarks-Baseline-518x600.png"
alt="Disk performance (various computers)" height="600" width="518"
title="Figure 2: Disk performance (various computers)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Infrastructure/Disk-Benchmarks-Baseline-823x954.png)

In my mind, the RAID 10 configuration on BEAST is still the "sweet spot" in
terms of performance, reliability, and cost. Keep in mind that you can probably
put together a 1 TB RAID 10 configuration today at a lower cost than I spent on
my 320 GB array many years ago. Heck, I bet you could probably put together a
600 GB RAID 10 array for about the same price as the SSD. [Of course, it's going
to consume significantly more energy than an SSD, but that's a price I'm willing
to pay.]

Also note that with the very large drives available today, you can get even
better (or perhaps I should say "more consistent") performance by using "short
stroking." However, if I were to go down that route for my home lab, I'd really
have to get my head examined.
