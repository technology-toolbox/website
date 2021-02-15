---
title: "Out-of-the-Box Permission (Policy) Levels"
date: 2007-04-23T16:57:00-07:00
excerpt: "For a couple of months now, I have been using the following command to add myself to a Microsoft Office SharePoint Server (MOSS) 2007 site restored from a different server: 
 stsadm.exe –o addpermissionpolicy –url http://foobar/sites/Migration -userlogin..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/04/23/out-of-the-box-permission-policy-levels.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/04/23/out-of-the-box-permission-policy-levels.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

For a couple of months now, I have been using the following command to add myself  to a Microsoft Office SharePoint Server (MOSS) 2007 site restored from a different  server:

```
stsadm.exe -o addpermissionpolicy -url
http://foobar/sites/Migration -userlogin 
{DOMAIN\username} -permissionlevel "Full Control"
```

This site is backed up from our Test environment (where the business users specify  the data) which happens to reside in a different domain. Consequently when the site  is restored, even the user restoring the site does not have permission to access  the site (that is, until running the above command).

Earlier today, I was helping another developer on our team debug a permissions  problem and we wanted to grant read-only access on the site to everyone.

I initially suggested the following command:

```
stsadm.exe -o addpermissionpolicy -url
http://foobar/sites/Migration -userlogin 
"NT AUTHORITY\Authenticated Users" -permissionlevel "Read"
```

However, we quickly discovered that "Read" wasn't quite right. After spending  no less than 10 minutes unsuccessfully trying variations -- such as "Read-only"  and "read only" -- and searching the Web for the documented list of available policies,  I ended up telling my colleague to just "punt" and use the UI instead.

Revisiting the issue a few hours later, I just spent a couple of minutes cranking  out a console application to help me understand the available options for the <kbd>permissionlevel</kbd> parameter:

```
static void Main(string[] args)
{
    Uri siteUrl = new Uri("http://foobar/sites/Migration");

    SPWebApplication application = SPWebApplication.Lookup(siteUrl);

    foreach (SPPolicyRole policyRole in application.PolicyRoles)
    {
        Console.WriteLine(policyRole.Name);
    }
}
```

The output is as follows:

<samp>Full Control<br>
Full Read<br>
Deny Write<br>
Deny All</samp>

Therefore the command that I should have suggested to my colleague is:

```
stsadm.exe -o addpermissionpolicy -url
http://foobar/sites/Migration -userlogin 
"NT AUTHORITY\Authenticated Users" -permissionlevel "Full Read"
```

