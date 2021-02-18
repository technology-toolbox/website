---
title: "Using Password Minder to Manage Your Passwords"
date: 2009-11-06T23:18:00-07:00
excerpt: "I started using Password Minder almost immediately after reading about it in the July 2004 edition of MSDN Magazine. I don't know about you, but trying to remember all of my various passwords for dozens of Web sites, numerous network accounts, VMs, etc..."
aliases: ["/blog/jjameson/archive/2009/11/06/using-password-minder-to-manage-your-passwords.aspx"]
draft: true
categories: ["My System"]
tags: ["My System", "Toolbox"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/11/07/using-password-minder-to-manage-your-passwords.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/11/07/using-password-minder-to-manage-your-passwords.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

I started using [Password Minder](http://msdn.microsoft.com/en-us/magazine/cc163958.aspx) almost immediately after reading about it in the July 2004 edition of MSDN Magazine. I don't know about you, but trying to remember all of my various passwords for dozens of Web sites, numerous network accounts, VMs, etc. is really quite a nightmare. [I'm actually a little embarrassed to say that on more than one occasion I've had to hack one of my VMs in order to reset the password for a domain admin. This process definitely isn't for the faint of heart, but it was better than having to rebuild [my FABRIKAM domain](/blog/jjameson/2009/09/13/the-jameson-datacenter)from scratch.]

Instead I tend to use no more than three or four passwords that I can remember (and change every couple of months) and a whole bunch of randomly generated passwords that I rely on Password Minder to store securely.

Note that all of the service accounts used in the ["Jameson Datacenter"](/blog/jjameson/2009/09/13/the-jameson-datacenter) (a.k.a. my home lab) use random 20 character passwords.

Hence, why I added Password Minder to my [Toolbox](/blog/jjameson/2007/03/21/backedup-and-notbackedup) years ago. [Thanks to Keith Brown for sharing this wonderful utility, as well as take the time to write about the internals of how it works.]

However, there were a few problems I had with the original Password Minder.

First, it didn't like the fact that my Documents folder is redirected to a server. In other words, my encrypted password file is:

> \\BEAST\Users$\jjameson\Documents\My Passwords.xml

Consequently, I encountered the following error when trying to save my password file:

{{< blockquote "font-italic text-danger" >}}

Failed to save the password file. Here are the gory details:

GetVolumeInformation failed:
The filename, directory name, or volume label syntax is incorrect.

{{< /blockquote >}}

I discovered that I could avoid this error by clearing the **Let Password Minder control the DACL** checkbox in the **Password Minder Options** dialog window.

The next problem that I encountered was Password Minder throwing up on me (in other words, generating an unhandled `IndexOutOfRangeException`) when one of my passwords somehow ended up being an empty string (to this day I'm still not sure how that empty password managed to get saved to the file).

Thus I made a small change to line 381 of CryptMaster.cs. The original code was:

```
if (null == record.EncryptedUserId) {
```

I changed this to:

```
if (string.IsNullOrEmpty(record.EncryptedUserId) == true) {
```

The third problem (which I didn't encounter until a few years later) was that Password Minder wouldn't run on my Windows Vista x64 desktop.

Note that the original project settings specified **Platform target: Any CPU** and thus pwm.exe would generate a [`BadImageFormatException`](http://msdn.microsoft.com/en-us/library/system.badimageformatexception.aspx) when trying to load the 32-bit NativeHelpers.dll.

{{< blockquote "font-italic text-danger" >}}

Could not load file or assembly 'NativeHelpers, Version=1.5.0.4, Culture=neutral, PublicKeyToken=null' or one of its dependencies. An attempt was made to load a program with an incorrect format.

{{< /blockquote >}}

This was easy enough to fix simply by setting **Platform target** to **x86** and then recompiling.

So, why do I blog about Password Minder after all this time? Well, just this week I finally got around to incorporating it into my "Toolbox" Visual Studio solution that is scheduled to build daily using Team Foundation Server, and I discovered that my Team Foundation Build server wasn't setup to build C++ projects, which is what I will blog about next.

Stay tuned...I need to eat some breakfast first ;-)

