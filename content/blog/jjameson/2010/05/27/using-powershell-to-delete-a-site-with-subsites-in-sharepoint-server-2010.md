---
title: "Using PowerShell to Delete a Site with Subsites in SharePoint Server 2010"
date: 2010-05-27T05:39:00-06:00
excerpt: "When using the \"DR.DADA\" approach to SharePoint development , I often find myself deleting sites (in DEV and TEST environments) and subsequently re-activating features or running some migration utility to recreate the site hierarchy. 
 In fact, a few..."
aliases: ["/blog/jjameson/archive/2010/05/26/using-powershell-to-delete-a-site-with-subsites-in-sharepoint-server-2010.aspx", "/blog/jjameson/archive/2010/05/27/using-powershell-to-delete-a-site-with-subsites-in-sharepoint-server-2010.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["SharePoint 2010", "PowerShell"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/27/using-powershell-to-delete-a-site-with-subsites-in-sharepoint-server-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/27/using-powershell-to-delete-a-site-with-subsites-in-sharepoint-server-2010.aspx)
>
> Since 		[I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog  		ever goes away.

When using the ["DR.DADA" approach to SharePoint development](/blog/jjameson/2009/03/31/introducing-the-dr-dada-approach-to-sharepoint-development), I often find myself deleting sites  (in DEV and TEST environments) and subsequently re-activating features or running  some migration utility to recreate the site hierarchy.

In fact, a few years ago this became such a common task on the Agilent Technologies  project that I wrote a simple "DeleteWeb" program to overcome the fact that {{< kbd "stsadm.exe -o deleteweb" >}} doesn't work on sites that have subsites.

Note that there really wasn't much to the DeleteWeb program. Almost all of the  work was performed by the **DeleteHelper** class:

```
using System;
using System.Diagnostics;
using System.Globalization;

using Microsoft.SharePoint;

namespace DeleteWeb
{
    public sealed class DeleteHelper
    {
        private DeleteHelper() { } // all members are static

        public static void DeleteWeb(
            string siteUrl)
        {
            using (SPSite site = new SPSite(siteUrl))
            {
                using (SPWeb web = site.OpenWeb())
                {
                    if (string.Compare(web.Url, siteUrl, true,
                        CultureInfo.InvariantCulture) != 0)
                    {
                        throw new InvalidOperationException(
                            "Invalid Web URL. Verify the URL and try again.");
                    }

                    DeleteWeb(web);
                }
            }
        }

        public static void DeleteWeb(
            SPWeb web)
        {
            SPWebCollection subwebs = web.GetSubwebsForCurrentUser();

            if (subwebs.Count > 0)
            {
                foreach (SPWeb subweb in subwebs)
                {
                    DeleteWeb(subweb);
                    subweb.Dispose();
                }
            }

            Debug.WriteLine("Deleting web: " + web.ServerRelativeUrl);
            web.Delete();
            return;
        }
    }
}
```

I was hoping that SharePoint Server 2010 would address this scenario out-of-the-box,  but that doesn't appear to be the case.

Suppose that you have a site (e.g. [http://foobar/Test](http://foobar/Test))  that has subsites (e.g. [http://foobar/Test/foo](http://foobar/Test/foo)  and [http://foobar/Test/bar](http://foobar/Test/bar)). If you attempt  to use the **[Remove-SPWeb](http://technet.microsoft.com/en-us/library/ff607890.aspx)**  cmdlet in SharePoint Server 2010 to delete the site...

```
Remove-SPWeb "http://foobar/Test" -Confirm:$false
```

...then you will encounter an error similar to the following:

{{< blockquote "font-italic text-danger" >}}

Remove-SPWeb : Error deleting Web site "/Test". You can't delete a site that  	has subsites.
At line:1 char:13
+ Remove-SPWeb &lt;&lt;&lt;&lt; "http://foobar/Test" -Confirm:$false
+ CategoryInfo : InvalidData: (Microsoft.Share...CmdletRemoveWeb:SPCmdletRemoveWeb)  	[Remove-SPWeb], SPException
+ FullyQualifiedErrorId : Microsoft.SharePoint.PowerShell.SPCmdletRemoveWeb

{{< /blockquote >}}

In order to delete a site that has subsites using PowerShell, we simply need  to convert the C# code shown above into a corresponding PowerShell function, as  shown below:

```
# Completely deletes the specified Web (including all subsites)

function RemoveSPWebRecursively(
    [Microsoft.SharePoint.SPWeb] $web)
{
    Write-Debug "Removing site ($($web.Url))..."

    $subwebs = $web.GetSubwebsForCurrentUser()

    foreach($subweb in $subwebs)
    {
        RemoveSPWebRecursively($subweb)
        $subweb.Dispose()
    }

    $DebugPreference = "SilentlyContinue"
    Remove-SPWeb $web -Confirm:$false
    $DebugPreference = "Continue"
}

$DebugPreference = "SilentlyContinue"
$web = Get-SPWeb "http://foobar/Test"
$DebugPreference = "Continue"

If ($web -ne $null)
{
    RemoveSPWebRecursively $web
    $web.Dispose()
}
```

Note that the script handles the case where `$web`  is null -- in other words, when the specified Web doesn't exist (for example, when  running the script a second time).

