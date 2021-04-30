---
title: "Moving the Virtual Hard Disk for Images (ext4.vhdx) in Docker Desktop"
date: 2021-04-30T11:39:05-06:00
excerpt:
  "Are you a little irritated that Docker Desktop is consuming a large chunk of
  your C: drive? Don't fret, you can easily resolve the issue -- provided you
  have another disk available."
categories: ["Development", "Infrastructure"]
tags: ["Docker"]
---

In the past, I've used Ubuntu and Minikube virtual machines for all Docker and
Kubernetes development. When I looked at Docker Desktop for Windows several
years ago, it just felt "lacking" compared to the other versions of Docker for
Mac and Linux.

However, this week I decided to revisit the setup of my development environments
by installing the latest version of
[Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/).
One of the first things I discovered is you probably want to move the virtual
hard disk used to store Docker images off the C: drive -- if, like me, you run a
relatively small SSD for your boot/system drive.

Note that on my primary development desktop (STORM), my C: drive is a Samsung
850 Pro 256 GB SSD -- which currently has approximately 17 GB of free space.
That's not a lot, but -- as long as I'm careful about what I store on system
drive -- it is sufficient for my needs. So, shortly after installing Docker
Desktop for Windows and running a few containers, I discovered the following:

{{< figure src="https://assets.technologytoolbox.com/screenshots/F3/5CCB258F8ECC8934921DD9F2FC5A3CBEDF7C91F3.png" alt="Figure 1: SharePoint development VM configuration" class="screenshot" height="1039" width="1918" title="Figure 1: Disk space consumed by Docker images" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/F3/5CCB258F8ECC8934921DD9F2FC5A3CBEDF7C91F3.png)

> **Note**
>
> The screenshot above was captured after temporarily moving most of the files
> in my **C:\NotBackedUp\GitHub** folder (approximately 6 GB) to a different
> disk. I only mention this because some readers might notice the amount of free
> space at the time of the screenshot was approximately 22 GB -- even with 8 GB
> of Docker images -- which doesn't match up with what I stated previously about
> currently having 17 GB of free space.

To avoid having Docker consume a large chunk on C: (again, an SSD), I moved the
virtual hard disk used for storing images to a "spinning pile of rust" (as Scott
Hanselman is fond of saying) -- a.k.a. the "F:" drive (Seagate 3 TB HDD).

Here are the steps I used to move the **ext4.vhdx** file (which is stored in a
**docker-desktop-data** folder inside the **Docker** folder highlighted in the
screenshot above). I previously published these steps in the
["STORM"](https://github.com/technology-toolbox/Notebook/blob/main/Infrastructure/STORM%20-%20Windows%2010%20Enterprise%20x64.md)
page within the
[shared notebook](https://github.com/technology-toolbox/Notebook) used to manage
the infrastructure for Technology Toolbox. I'm copying them here to make them
more discoverable and easier to digest.

## Move Docker images from C: drive (SSD) to F: drive (HDD)

### Quit Docker Desktop

In the Windows system tray, right-click the Docker icon and click **Quit Docker
Desktop**.

{{< div-block "note important" >}}

> **Important**
>
> Ensure Docker Desktop is not running before proceeding to the next step.

{{< /div-block >}}

### Move Docker Desktop storage (VHDX) used for images

Open a PowerShell window and run the following commands to move the virtual hard
disk used by Docker Desktop to store images:

```PowerShell
wsl --shutdown

wsl --export docker-desktop-data docker-desktop-data.tar

wsl --unregister docker-desktop-data

mkdir F:\NotBackedUp\jjameson\docker-desktop-data

wsl --import docker-desktop-data F:\NotBackedUp\jjameson\docker-desktop-data `
    .\docker-desktop-data.tar --version 2
```

> **Note:**
>
> I chose to store the VHDX inside a "user" folder on the F: drive, since the
> file originally resided in the **AppData** folder for my account on the C:
> drive.

### References

- [How to move ext4.vhdx to a non system disk?](https://github.com/docker/for-win/issues/7348)
- [how to move the vhdx of wsl2 to other disk](https://github.com/MicrosoftDocs/WSL/issues/412)
