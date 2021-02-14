---
title: "Disabling Hibernation in Windows Vista (and Deleting Hiberfil.sys)"
date: 2008-02-17T14:33:00+08:00
excerpt: "As I described in my previous post , I found myself with a paltry 320 MB of free space on a 20 GB partition after installing Windows Vista and a handful of programs (which although I specified to install on a different partition, ended up \"bloating\" my..."
draft: true
categories: ["Infrastructure"]
tags: ["Windows Vista"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:
> 
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2008/02/17/disabling-hibernation-in-windows-vista-and-deleting-hiberfil-sys.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/02/17/disabling-hibernation-in-windows-vista-and-deleting-hiberfil-sys.aspx)
> 
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.


As I described in [my previous post](/blog/jjameson/2008/02/17/an-update-on-disk-space-usage-by-windows-vista), I found myself with a paltry 320 MB of free space on a         20 GB partition after installing Windows Vista and a handful of programs (which         although I specified to install on a different partition, ended up "bloating" my         system partition to an unmanageable point).

In order to free up some much needed space, I decided to disable hibernation, which         would seem like a straightforward process. In fact, if memory serves, this was actually         quite easy in Windows XP. However, in Vista, disabling hibernation **and**         deleting the corresponding system file is a little tricky. Note that on my laptop         with 4 GB of RAM, C:\hiberfil.sys consumes about 3.37 GB -- which I really need         back, thank you very much.

My initial assumption was that the option to disable hibernation lies somewhere         in the **Power Options **area of the **Control Panel**.         However, after poking around in there for a few minutes and not finding it, I proceeded         to perform a search in **Control Panel **for "**hibernate**"         which displayed the following result:


> **Turn hibernation on or off**


Perfect...this looks like precisely the option I was looking for. Unfortunately,         when I clicked this link, it just took me back to the **Power Options **         area that I had already thoroughly explored. Figuring I had simply missed the obvious,         I proceeded to set the following options under advanced power settings:


> **Sleep**
> 
> 
> > **Hibernate after**
> 
> 
> 
> > > **On battery: Never**
> > > 
> > > **Plugged in: Never**


Note that setting the value to **0 **minutes is the same as saying         "never hibernate." Okay, so now I should be able to delete the massive hiberfil.sys         file, right?

Wrong.

Okay, so I guess Vista still has a lock on the file since I haven't rebooted since         I "disabled" hibernation. Fine, I'll reboot my laptop. Now, I should finally be         able to delete hiberfil.sys, right?

Still wrong.

Come on now...be reasonable...what's it going to take to delete that file?

It turns out that the way you delete hiberfil.sys is by using the **Disk Cleanup
        **utility.

To delete hiberfil.sys in Windows Vista:

1. Start **Disk Cleanup**.
2. On the **Disk Cleanup Options **window, click **Files from all users
                    on this computer**.
3. On the **Disk Cleanup: Drive Selection **window, click select the system
                    drive where Windows Vista is installed, and then click **OK**.
4. Wait for the utility to scan the drive and then in the **Disk Cleanup **
                    window, click the checkbox next to **Hibernation File Cleaner**, clear
                    all of the other checkboxes, and then click **OK**. When prompted to
                    confirm the operation, click **Delete Files**.


After performing these steps, the free space on my Windows Vista partition (C:)         increased from 320 MB to 3.66 GB. Woohoo!

That should get me going for a while -- or at least until I decide to install a         few more programs on E: ;-)

