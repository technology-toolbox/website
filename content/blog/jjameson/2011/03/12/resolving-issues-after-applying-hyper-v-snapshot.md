---
title: Resolving Issues After Applying Hyper-V Snapshot
date: 2011-03-12T04:04:00-07:00
excerpt:
  "This morning I rolled back one of my development VMs to a snapshot I created
  about a month ago. When I subsequently tried to login with my domain
  credentials, I encountered the following error: The trust relationship between
  this workstation and the..."
aliases:
  [
    "/blog/jjameson/archive/2011/03/11/resolving-issues-after-applying-hyper-v-snapshot.aspx",
    "/blog/jjameson/archive/2011/03/12/resolving-issues-after-applying-hyper-v-snapshot.aspx",
  ]
draft: true
categories: ["Infrastructure"]
tags: ["Windows Server", "Infrastructure", "Virtualization"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/03/12/resolving-issues-after-applying-hyper-v-snapshot.aspx"
---

This morning I rolled back one of my development VMs to a snapshot I created
about a month ago. When I subsequently tried to login with my domain
credentials, I encountered the following error:

{{< blockquote "fst-italic text-danger" >}}

The trust relationship between this workstation and the primary domain failed.

{{< /blockquote >}}

I did a quick Internet search for this error and found the following KB article:

{{< reference title="Trust Relationship Between Workstation and Domain Fails"
linkHref="http://support.microsoft.com/kb/162797" >}}

While you _could_ follow the instructions in KB 162797 to resolve this error
(removing the computer from the domain and then adding it back), there's a much
easier way to resolve the error:

{{< reference
title="How to use Netdom.exe to reset machine account passwords of a Windows Server domain controller"
linkHref="http://support.microsoft.com/kb/325850" >}}

Don't be put off by the title of this KB article. You just need to read a little
bit into it:

{{< blockquote "fst-italic" >}}

[...] This procedure is most frequently used on domain controllers, but also
applies to any Windows machine account.

{{< /blockquote >}}

To resolve the error after applying an old Hyper-V snapshot on a VM joined to a
domain:

1. Login to the VM using a local administrator account.

2. Open an administrator command prompt and run the following command:
   
   {{< console-block-start >}}
   
   netdom resetpwd /s:{server} /ud:{DOMAIN\user} /pd:\*
   
   {{< console-block-end >}}
   For example:
   
   {{< console-block-start >}}
   
   netdom resetpwd /s:XAVIER1 /ud:TECHTOOLBOX\jjameson /pd:\*
   
   {{< console-block-end >}}
   
   {{< div-block "note" >}}
   
   > **Note**
   > 
   > XAVIER1 is one of the domain controllers in my home lab (TECHTOOLBOX).
   
   {{< /div-block >}}

3. Logout and log back in using a domain account.

Note that this issue doesn't always occur when rolling back a snapshot. It
depends on how old the snapshot is (specifically whether or not the machine
account password has changed in the domain since the snapshot was taken).

One of the other things I've learned about using snapshots with a domain-joined
VM is that you should be sure to enable the **Time synchronization** service on
the VM. I typically disable this service on domain-joined VMs (since the time is
synchronized from the domain controller). However, when using snapshots, the
latency in waiting for the time to synchronize after applying a snapshot can
quickly become unbearable (especially if you are frequently applying a
snapshot).
