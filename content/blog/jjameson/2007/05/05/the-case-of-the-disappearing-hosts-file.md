---
title: The Case of the Disappearing Hosts File
date: 2007-05-05T09:18:00-06:00
excerpt:
  Hmmm...how should I phrase this? It has been a very educational couple of
  weeks on my current SharePoint project. During the rebuild of our Test
  environment, the SharePoint Products and Technologies Configuration Wizard
  failed when it was unable...
aliases:
  [
    "/blog/jjameson/archive/2007/05/04/the-case-of-the-disappearing-hosts-file.aspx",
    "/blog/jjameson/archive/2007/05/05/the-case-of-the-disappearing-hosts-file.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/05/05/the-case-of-the-disappearing-hosts-file.aspx"
---

Hmmm...how should I phrase this?

It has been a very _educational_ couple of weeks on my current SharePoint
project.

During the rebuild of our Test environment, the SharePoint Products and
Technologies Configuration Wizard failed when it was unable to find the
**%SystemRoot%\System32\drivers\etc\hosts** file. We had encountered this error
during the original installation on the SSP server in the Test environment
because someone had renamed the file from **hosts** to **hosts-old**. Therefore
we suspected the same problem this time around, thinking that perhaps there was
some scheduled script or group policy that was disabling local host name
resolution.

For those of you that may not have attempted to read the status messages as they
flash by in the Configuration Wizard, step 4 changes the permissions on the
hosts file to grant the WSS_ADMIN_WPG group the following permissions:

- List Folder / Read Data
- Read Attributes
- Read Extended Attributes
- Create Files / Write Data
- Create Folders / Append Data
- Write Attributes
- Write Extended Attributes
- Delete
- Read Permissions

After recreating the hosts file, I was able to successfully complete the
Configuration Wizard (because the security settings on the hosts files could now
be set in step 4). However, a short time later, I noticed the following in the
event log:

```Text
Application Server Administration job failed for service instance
 Microsoft.Office.Server.Search.Administration.SearchServiceInstance (...).

Reason: Could not find file 'D:\WINNT\system32\drivers\etc\HOSTS'.

Techinal Support Details:
System.IO.FileNotFoundException: Could not find file 'D:\WINNT\system32\drivers\etc\HOSTS'.
File name: 'D:\WINNT\system32\drivers\etc\HOSTS'
   at System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
   at System.IO.FileStream.Init(...)
   at System.IO.FileStream..ctor(...)
   at System.IO.StreamReader..ctor(...)
   at System.IO.FileInfo.OpenText()
   at Microsoft.Search.Administration.Security.HOSTSFile.ParseHOSTSFile(...)
   at Microsoft.Search.Administration.Security.HOSTSFile.ConfigureDedicatedGathering(...)
   at Microsoft.Office.Server.Search.Administration.SearchServiceInstance.SynchronizeDefaultContentSource(...)
   at Microsoft.Office.Server.Search.Administration.SearchServiceInstance.Synchronize()
   at Microsoft.Office.Server.Administration.ApplicationServerJob.ProvisionLocalSharedServiceInstances(...)
```

Argh! The hosts file has disappeared again!

I restored the hosts file again and set the permissions manually for the
WSS_ADMIN_WPG group. However I noticed that it quickly disappeared again.

I then restored the hosts file (yet) again, but did not give the WSS_ADMIN_WPG
group permission to delete the file. This resulted in the following event log
entry:

```Text
Application Server Administration job failed for service instance
 Microsoft.Office.Server.Search.Administration.SearchServiceInstance (...).

Reason: Access to the path 'D:\WINNT\system32\drivers\etc\HOSTS' is denied.

Techinal Support Details:
System.UnauthorizedAccessException: Access to the path 'D:\WINNT\system32\drivers\etc\HOSTS' is denied.
   at System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
   at System.IO.FileInfo.Delete()
   at Microsoft.Search.Administration.Security.HOSTSFile.CleanupDedicatedGathering(...)
   at Microsoft.Search.Administration.Security.HOSTSFile.ConfigureDedicatedGathering(...)
   at Microsoft.Office.Server.Search.Administration.SearchServiceInstance.SynchronizeDefaultContentSource(...)
   at Microsoft.Office.Server.Search.Administration.SearchServiceInstance.Synchronize()
   at Microsoft.Office.Server.Administration.ApplicationServerJob.ProvisionLocalSharedServiceInstances(...)
```

Aha! So apparently the guilty party for deleting my hosts file isn't some
malicious system administrator or group policy setting, but rather it is the
Windows SharePoint Services Timer itself!

It turns out that this is a bug in MOSS 2007 (although I am still waiting for
PSS to formally acknowledge that this is a bug). Nevertheless, I am convinced
that this is a bug. Here's why:

I never recommend to customers having your service accounts be members of the
local Administrators group. Quite honestly, this is simply too dangerous and
presents a whole slew of risks that are beyond the scope of this posting. Since
the service account that I am using is _not_ a member of the local
Administrators group, when the timer job deletes the file, it does not have
permission to recreate the file. Recall that earlier I mentioned that step 4 of
the Configuration Wizard only grants permissions on the hosts file itself to the
WSS_ADMIN_WPG group (which the service account is a member of). Hence the
disappearing hosts file.

The workaround is to grant the following permissions for the WSS_ADMIN_WPG on
the **%SystemRoot%\System32\drivers\etc** folder:

- Traverse Folder / Execute File
- List Folder / Read Data
- Read Attributes
- Read Extended Attributes
- Create Files / Write Data
- Read Permissions

For those of you that may be wondering why SharePoint needs access to the hosts
file at all, the answer is due to one of the configuration settings that you can
specify for **Office SharePoint Server Search**.

In our case, we are using a farm configuration similar to what used to be
referred in SPS 2003 as a "Medium farm" -- i.e. two front-end Web servers and
one SSP server (previously referred to as the job/index server). In order to
minimize the impact on users, we have the index server (i.e. the SSP) crawl
itself.

In Central Administration, if you specify a dedicated front-end for crawling
contant, then a timer job is created to add an entry to your hosts file to force
the Web application (i.e. the host header) to resolve to that server.
Unfortunately, instead of "editing" the file in-place, the developer who
implemented this feature decided it would be easier to just read the hosts file,
add the appropriate entries, delete the original file, and then create a new
file. If your SharePoint farm account (i.e. the one that the Windows SharePoint
Servers Timer runs under) is a member of the local Administrators group then
there is no problem (which appears to be how this feature was tested). However,
if you adhere to the "principle of least privilege" then there's definitely a
problem.

Unfortunately we did not discover the bug with the disappearing hosts file in
DEV because we are using a single server configuration there (and therefore did
not specify a dedicated Web front-end for crawling).

I'll be writing a few more blogs posts this weekend to describe a couple of the
other "gotchas" I have discovered in the last couple of weeks, but right now it
is time for some pancakes!
