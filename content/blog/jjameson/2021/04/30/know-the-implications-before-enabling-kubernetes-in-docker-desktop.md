---
title: "Know the Implications Before Enabling Kubernetes in Docker Desktop"
date: 2021-04-30T14:19:09-06:00
description:
  Kubernetes is incredibly powerful, but should running it be as simple as
  selecting a checkbox?
images: ["https://assets.technologytoolbox.com/screenshots/1A/91DB918AABE5855CA91741DFEED8E25B38D7761A.png"]
categories: ["Development", "Infrastructure"]
tags: ["Docker", "Kubernetes"]
---

As noted in my
[previous post](/blog/jjameson/2021/04/30/moving-the-virtual-hard-disk-for-images-ext4-vhdx-in-docker-desktop/),
this week I decided to revisit the setup of my development environments by
installing the latest version of
[Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/).

Shortly after installing Docker Desktop, I enabled Kubernetes as shown in the
following screenshot.

{{< figure src="https://assets.technologytoolbox.com/screenshots/1A/91DB918AABE5855CA91741DFEED8E25B38D7761A.png" class="screenshot" height="720" width="1250" title="Figure 1: Enabling Kubernetes in Docker Desktop" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/1A/91DB918AABE5855CA91741DFEED8E25B38D7761A.png)

As I described in my previous post, I also moved the virtual hard disk that
Docker uses to store images from my C: drive (SSD) to my F: drive (HDD). Note
that my F: drive is used as "secondary" storage. I don't expect this to be
anywhere near as fast as my "primary" storage. In fact, the volume label for the
F: drive is **Bronze01**, as you can see in the screenshot below.

{{< figure src="https://assets.technologytoolbox.com/screenshots/00/3598C3176A030CDB8BFA36F9155101C721B91300.png" class="screenshot" height="466" width="1011" title="Figure 2: Storage configuration on Windows 10 desktop (STORM)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/00/3598C3176A030CDB8BFA36F9155101C721B91300.png)

One of the things I noticed after installing Docker Desktop and enabling
Kubernetes is that my F: drive _never_ stopped spinning.

> **Note**
>
> Like most HDDs, my F: drive makes a little bit of noise when it is active.
> Consequently, I can audibly detect whenever the drive spins up to read or
> write data. Usually this drive is idle because, as I mentioned before, this is
> "secondary" storage.

Even after coming back from lunch or a few hours away from the computer, I could
tell just by the sound that my typically idle HDD was still very much active.

I fired up Performance Monitor to confirm my hypothesis that the new behavior
was due to Docker Desktop and, in particular, to enabling Kubernetes.

{{< figure src="https://assets.technologytoolbox.com/screenshots/0F/5B8FA0E2895D56F6F3D60DC32272309C2C3DB00F.png" class="screenshot" height="1040" width="1286" title="Figure 3: Performance monitor for Docker Desktop and Kubernetes" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/0F/5B8FA0E2895D56F6F3D60DC32272309C2C3DB00F.png)

Before I started capturing metrics in Performance Monitor, I stopped all of the
containers that I had previously created.

As you can see, there is quite a bit of processor activity when Kubernetes is
enabled in Docker Desktop. At approximately 8:42 AM, I stopped Docker Desktop --
which obviously stops all of the system containers (a.k.a. "Kubernetes internal
containers") as well -- and you can see where the disk activity for the F: drive
flatlined.

At approximately 8:50 AM, I restarted Docker Desktop and you can see the "flurry
of activity" due to a number of my containers restarting. I waited a few minutes
and then stopped all non-system containers. As expected, resource utilization
returned to where it was before (i.e. still a signficant amount of processor
utilization and an HDD that is always active -- even when you might think the
system would be idle).

Shortly before 9:08 AM, I disabled Kubernetes in Docker Desktop -- but kept it
running in the background. Before the data showed up in Performance Monitor, I
could tell just by the sound that the situation had improved substantially.
Notice that with Kubernetes disabled but Docker Desktop still running, there is
no disk activity on the F: drive. None! Zip! Zilch! Nada!

So, what's the lesson here? Well, just like the title says, you really need to
know the implications before enabling Kubernetes in Docker Desktop. I'm
certainly not saying that you shouldn't enable it. However, you need to be aware
that doing so will introduce a continuous load on your machine.

Perhaps, like me, you have been running Docker or Minikube using Linux virtual
machines stored on SSDs -- rather than via Docker Desktop for Windows. Given the
nature of running VMs, I know when I explicitly start and stop them -- so, even
if I don't hear any hard disk activity -- since I typically store these on my D:
drive (another SSD) -- I'm still aware of when they are running and I typically
shut them down when I'm not using them.

If you Google something like
[_docker laptop battery_](https://www.google.com/search?q=docker+laptop+battery),
you'll find lots of people out there complaining how Docker is draining their
MacBook or laptop batteries or how the
[vmmem](https://devblogs.microsoft.com/oldnewthing/20180717-00/?p=99265) process
used by Docker on Windows is consuming too much CPU.

Maybe -- and I must emphasize _maybe_ since I don't know their specific setups
-- it is not Docker but rather Kubernetes that is the culprit.

## References

- [Energy Usage on OSX #4323](https://github.com/docker/for-mac/issues/4323)
- [Windows 10 Docker processes consuming high CPU with no running containers #1772](https://github.com/docker/for-win/issues/1772)
- [Resource usage of docker with no containers running?](https://superuser.com/questions/1430261/resource-usage-of-docker-with-no-containers-running)
