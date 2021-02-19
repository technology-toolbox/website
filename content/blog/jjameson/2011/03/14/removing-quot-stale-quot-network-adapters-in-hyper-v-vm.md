---
title: "Removing \"Stale\" Network Adapters in Hyper-V VM"
date: 2011-03-14T04:13:00-06:00
excerpt: "Each time I create (or recreate) a virtual machine in Hyper-V using one of my SysPrep'ed images , I usually end up having to do a quick Internet search for: 
 device manager show hidden devices 
 I'm a little embarrassed to say that I simply can't remember..."
aliases: ["/blog/jjameson/archive/2011/03/13/removing-quot-stale-quot-network-adapters-in-hyper-v-vm.aspx", "/blog/jjameson/archive/2011/03/14/removing-quot-stale-quot-network-adapters-in-hyper-v-vm.aspx"]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Infrastructure", "Virtualization"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/14/removing-quot-stale-quot-network-adapters-in-hyper-v-vm.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/14/removing-quot-stale-quot-network-adapters-in-hyper-v-vm.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

Each time I create (or recreate) a virtual machine in Hyper-V using [one of my SysPrep'ed images](/blog/jjameson/2009/08/13/using-sysprep-ed-vhds-for-new-hyper-v-virtual-machines), I usually end up having to do a quick Internet  search for:

{{< blockquote "font-italic" >}}

device manager show hidden devices

{{< /blockquote >}}

I'm a little embarrassed to say that I simply can't remember the environment  variable that I need to set in order to remove what I call "stale" network adapters  in Device Manager.

Note that if you don't properly export/import a VM in Hyper-V (and instead simply  copy VHDs around like I often do), then you'll end up with a network adapter named  something like ""Microsoft Virtual Machine Bus Network Adapter #2" (and a hidden  network adapter named "Microsoft Virtual Machine Bus Network Adapter").

I'm not aware of any issues by *not* deleting these network adapters,  but I still like to clean them up anyway.

Here is the process that I use to cleanup the network adapters:

1. Start an administrator command prompt and then run the following two commands:
   
   ```
   set devmgr_show_nonpresent_devices=1
   start devmgmt.msc
   ```

2. In the **Device Manager**window:
   
   1. Click the **View** menu and then click **Show hidden
      devices**.
   2. Expand **Network adapters**.
   3. Right-click each network adapter that begins with **Microsoft
      Virtual Machine Bus Network Adapter** (e.g. "Microsoft Virtual Machine
      Bus Network Adapter", "Microsoft Virtual Machine Bus Network Adapter #2")
      and click **Uninstall**. When prompted to confirm the device
      uninstall, click **OK**.
   4. If you notice any extra **Microsoft ISATAP Adapter** items,
      then uninstall those as well. (I occasionally see these, but it doesn't
      happen all the time like the duplicate VM Bus network adapters.)
   5. Right-click **Network adapters** and then click **Scan for hardware changes**. (This will recreate the default adapter
      named "Microsoft Virtual Machine Bus Network Adapter".)

You should now have exactly one VM Bus network adapter (and one Microsoft ISATAP  Adapter).

Also, just in case it's not completely obvious, you should do this cleanup before  configuring any network settings like DNS servers or a static IP address.

Perhaps if the environment variable was <var>devmgmt_show_hidden_devices</var>  (instead of <var>devmgr_show_nonpresent_devices</var>) then I could actually remember  it ;-)

