---
title: "Event ID 10016, KB 920783, and the WSS_WPG Group"
date: 2009-10-16T21:33:00-07:00
excerpt: "If you've ever deployed Windows SharePoint Services (WSS) v3 or Microsoft Office SharePoint Server (MOSS) 2007 in a least privilege configuration, you have undoubtedly encountered errors similar to the following in your Windows event log: 
 The application..."
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "Simplify", "MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/17/event-id-10016-kb-920783-and-the-wss-wpg-group.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/17/event-id-10016-kb-920783-and-the-wss-wpg-group.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

If you've ever deployed Windows SharePoint Services (WSS) v3 or Microsoft
Office SharePoint Server (MOSS) 2007 in a least privilege configuration, you
have undoubtedly encountered errors similar to the following in your Windows
event log:

{{< blockquote "font-italic text-danger" >}}

    The application-specific permission settings do not grant Local Activation 
    permission for the COM Server application with CLSID {61738644-F196-11D0-9953-00C04FD919C1} 
    to the user TECHTOOLBOX\svc-sharepoint-dev SID (S-1-5-21-3914637029-2275272621-3670275343-1145) 
    from address LocalHost (Using LRPC). This security permission can be modified 
    using the Component Services administrative tool.

{{< /blockquote >}}

Specifically, I am referring to Event ID 10016 and 10017.

While these errors are apparently benign, it is still rather annoying to
have your event logs flooded with this kind of "noise." [Personally, I very
much like to see servers come up "clean" after a reboot (meaning no new errors
or warnings in the event log). This is especially true when you use something
like
[Systems Center Operations Manager (SCOM)](http://www.microsoft.com/systemcenter/operationsmanager/en/us/default.aspx) to monitor your servers, and one
or more members of the Operations team receives notification whenever an error
occurs on a Production server.]

There are lots of sources out on the Internet that provide the steps necessary
to resolve these errors (essentially you need to grant the SharePoint service
account Local Activation permission to the IIS WAMREG Admin Service).

This week, I was glad to discover that Microsoft has a KnowledgeBase article
([KB 920783](http://support.microsoft.com/kb/920783)) describing
the errors and the associated workaround. [This KB article appears to have been
published over two years ago, but honestly, I was completely unaware of it until
just yesterday when I was building out the Production MOSS 2007 environment
on my latest project.]

However, I see two problems with KB 920783:

- First, it doesn't mention Microsoft Office SharePoint Server (MOSS)
  2007 -- which might explain why I never came across this KB before when
  searching the Internet. Sure, we all know that MOSS 2007 is built on top
  of WSS v3 and thus KB articles that apply to "Microsoft Windows SharePoint
  Services 3.0" almost always apply to "Microsoft Office SharePoint Server
  2007", but if you don't specifically state that, you shouldn't expect Bing
  or Google to return the KB article when searching for something like "MOSS
  10016" or "Microsoft Office SharePoint Server 10016". Right?
- Second, and much more important, the KB article instructs you to:

{{< blockquote "font-italic" >}}

    "type the domain user account that you specified as the Windows SharePoint 
    Services 3.0 service account"

{{< /blockquote >}}

I definitely don't recommend doing that -- unless you just like making more
work for yourself than necessary. Instead, you should specify the local
**WSS\_ADMIN\_WPG** and **WSS\_WPG** groups rather than
a specific user. This is what I've been doing for the last couple of years so
trust me, it works.

The reason why I recommend using the "Windows SharePoint Services Worker
Process Group" instead of individual users is that you are likely (or at least
hopefully) using different service accounts for your SharePoint farm (e.g. TECHTOOLBOX\svc-sharepoint)
and for the application pools behind your Web applications (e.g. TECHTOOLBOX\svc-web-fabrikam).
Furthermore, if you are doing a large enterprise deployment of SharePoint, you
likely have multiple Web applications (e.g. SSP, My Sites, etc.) that each utilize
a separate service account.

Rather than granting the Local Activation permission to each of these service
accounts individually, it is much more effective to just grant the permission
to the WSS\_ADMIN\_WPG and WSS\_WPG groups instead.

> **Note**
>
> When creating a new Web application, SharePoint automatically adds the corresponding service account to the local WSS\_WPG group on each SharePoint server in your farm.

> **Update (2009-10-29)**
>
> Matt McEvoy contacted me last Monday regarding the fact that I didn't
> specify the **WSS\_ADMIN\_WPG** group -- only the **WSS\_WPG** group.
>
> Ugh...that will teach me to try to recall something like this from
> memory. When I was writing this blog post, I mistakenly thought that
> the service account for the SharePoint farm was added to both WSS\_ADMIN\_WPG
> and WSS\_WPG. However, this isn't the case.
>
> Therefore you need to be sure to apply the steps in
> [KB 920783](http://support.microsoft.com/kb/920783) using
> both groups if you want to rid your event logs of these errors once
> and for all. Again, this is assuming you are using least privilege accounts
> -- which I certainly hope you are.
>
> Thanks, Matt, for pointing out my omission.

> **Update (2010-05-03)**
>
> If performing this step on Windows Server 2008 R2, you must first take ownership of the corresponding registry key and grant Administrators permissions to update the configuration.
>
> To take allow the configuration of the IIS WAMREG Admin Service to
> be changed using the Component Services console:
>
> 1. Click the **Start** menu, type **regedit**,
>    and then click **regedit.exe**. If prompted by
>    **User Account Control** to allow the program to make
>    changes to this computer, click **Yes**.
> 2. In the **Registry Editor** window, search for "61738644-F196-11D0-9953-00C04FD919C1"
>    to find HKEY\_CLASSES\_ROOT\AppID\{61738644-F196-11D0-9953-00C04FD919C1}.
> 3. Right-click on the **HKEY\_CLASSES\_ROOT\AppID\{61738644-F196-11D0-9953-00C04FD919C1}**
>    key and then click **Permissions**.
> 4. In the **Permissions for {61738644-F196-11D0-9953-00C04FD919C1}**
>    dialog box, click **Advanced**.
> 5. In the **Advanced Security Settings for {61738644-F196-11D0-9953-00C04FD919C1}**dialog box:
>    1. Click the **Owner** tab.
>    2. In the **Change owner to** list, click the
>       **Administrators** group.
>    3. Click **OK**.
> 6. In the **Permissions for {61738644-F196-11D0-9953-00C04FD919C1}**
>    dialog box, click the **Administrators** group, then
>    click the checkbox to allow the group **Full Control**,
>    and click **OK**.
> 7. Close the Registry Editor window.
>
> Now that the Administrators group has sufficient permissions, follow
> the steps in KB 920783 to make the changes to the IIS WAMREG Admin Service.

