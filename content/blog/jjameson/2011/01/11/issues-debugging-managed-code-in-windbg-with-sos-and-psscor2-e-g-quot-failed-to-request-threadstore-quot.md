---
title: 'Issues Debugging Managed Code in WinDbg with SOS and PSSCOR2 (e.g. "Failed to request ThreadStore")'
date: 2011-01-11T05:42:00-07:00
description: 'Yesterday I found myself back in "WinDbg-land" after a long, long time
  (since 99% of my debugging is performed in development environments using
  Visual Studio). However, I couldn''t get the managed code debugging to work in
  WinDbg. I initially tried...'
aliases:
  [
    "/blog/jjameson/archive/2011/01/10/issues-debugging-managed-code-in-windbg-with-sos-and-psscor2-e-g-quot-failed-to-request-threadstore-quot.aspx",
    "/blog/jjameson/archive/2011/01/11/issues-debugging-managed-code-in-windbg-with-sos-and-psscor2-e-g-quot-failed-to-request-threadstore-quot.aspx",
  ]
categories: ["Development"]
tags: ["Debugging"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/01/11/issues-debugging-managed-code-in-windbg-with-sos-and-psscor2-e-g-quot-failed-to-request-threadstore-quot.aspx"
---

Yesterday I found myself back in "WinDbg-land" after a long, long time (since
99% of my debugging is performed in development environments using Visual
Studio).

However, I couldn't get the managed code debugging to work in WinDbg. I
initially tried SOS and later PSSCOR2, but they both refused to produce anything
even remotely helpful.

For example, when I ran the "!threads" command, WinDbg simply reported the
following:

{{< console-block-start >}}

{{< sample-output "Failed to request ThreadStore" >}}

{{< console-block-end >}}

Similarly, when I ran "!eeheap", the following message was displayed:

{{< console-block-start >}}

{{< sample-output "Unable to get system domain info" >}}

{{< console-block-end >}}

I have to admit, I was completely stumped. My initial research on the "Failed to
request ThreadStore" error suggested that this problem occurs when the symbols
are not loaded. However, even after purging my local symbol cache and
downloading fresh copies from
[http://msdl.microsoft.com/download/symbols](http://msdl.microsoft.com/download/symbols),
I still got the same errors.

Consequently I sent an email out last night to one of the debugging groups.
Fortunately, I didn't have to wait very long for a response.

This morning, Sukesh Ashok Kumar, a Support Escalation Engineer in the PSS
group, told me to check a few things:

- Use "lm v m mscorwks" and see if the symbols are loaded
- Use .cordll and see if the right mscordacwks is loaded
- Use correct architecture of extension sos/psscor2

As I mentioned before, I purged my symbol cache yesterday and downloaded fresh
copies from the Microsoft public symbol server, so the first suggestion didn't
seem to be the problem. However, I ran "ld mscorwks" anyway and verified the
symbols were loaded as expected.

Similarly, since I had originally used ".loadby sos mscorwks" to load the SOS
extension, I didn't think the third suggestion was the problem. Also note that
yesterday I downloaded a fresh copy of PSSCOR2 from the microsoft.com site and
copied the x86 version to the same folder as WinDbg (since I was debugging a
32-bit environment). Therefore it didn't seem like this was the issue either.

Per Sukesh's second suggestion, I ran the ".cordll" command, which reported the
following:

{{< console-block-start >}}

{{< sample-output "CLR DLL status: No load attempts" >}}

{{< console-block-end >}}

Note that I had never used this command before.

I then looked up ".cordll" in the debugging help file and read a little more
about it. I subsequently ran ".cordll -e" -- which showed the .NET Framework 4.0
mscordacwks.dll. A-ha!

That certainly wasn't right since I was debugging an ASP.NET worker process
running Microsoft Office SharePoint Server 2007. In other words, it needed to be
the .NET Framework 2.0 version of mscordacwks.dll.

I then restarted WinDbg, attached to the IIS worker process, and ran the
following:

```WinDbg
0:023> {{< kbd "ld mscorwks" >}}
{{< sample-output "Symbols loaded for mscorwks" >}}
0:023> {{< kbd ".cordll -lp C:\Windows\Microsoft.NET\Framework\v2.0.50727" >}}
{{< sample-output "CLR DLL status: No load attempts" >}}
0:023> {{< kbd "!threads" >}}
Index TID   TEB    StackBase   StackLimit   DeAlloc   StackSize   ThreadProc
0 00000dec 0x7ffdf000 0x00110000 0x00105000 0x000d0000 0x0000b000 0x0
1 0000156c 0x7ffde000 0x01550000 0x0154e000 0x01510000 0x00002000 0x0
2 0000144c 0x7ffdd000 0x00df0000 0x00dee000 0x00db0000 0x00002000 0x0
...
```

Woohoo! (As you can see, the "!threads" command now works as expected.)

Apparently the installation of .NET Framework 4.0 (via Windows Update) has
necessitated a change in the debugging process.
