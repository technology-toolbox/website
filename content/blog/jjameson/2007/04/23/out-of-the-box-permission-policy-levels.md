---
title: Out-of-the-Box Permission (Policy) Levels
date: 2007-04-23T22:57:00-06:00
description:
  "For a couple of months now, I have been using the following command to add
  myself to a Microsoft Office SharePoint Server (MOSS) 2007 site restored from
  a different server: stsadm.exe –o addpermissionpolicy –url
  http://foobar/sites/Migration -userlogin..."
aliases:
  [
    "/blog/jjameson/archive/2007/04/23/out-of-the-box-permission-policy-levels.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/04/23/out-of-the-box-permission-policy-levels.aspx"
---

For a couple of months now, I have been using the following command to add
myself to a Microsoft Office SharePoint Server (MOSS) 2007 site restored from a
different server:

{{< console-block >}}

stsadm.exe -o addpermissionpolicy -url
[http://foobar/sites/Migration](http://foobar/sites/Migration) -userlogin
{DOMAIN\username} -permissionlevel "Full Control"

{{< /console-block >}}

This site is backed up from our Test environment (where the business users
specify the data) which happens to reside in a different domain. Consequently
when the site is restored, even the user restoring the site does not have
permission to access the site (that is, until running the above command).

Earlier today, I was helping another developer on our team debug a permissions
problem and we wanted to grant read-only access on the site to everyone.

I initially suggested the following command:

{{< console-block >}}

stsadm.exe -o addpermissionpolicy -url
[http://foobar/sites/Migration](http://foobar/sites/Migration) -userlogin "NT
AUTHORITY\Authenticated Users" -permissionlevel "Read"

{{< /console-block >}}

However, we quickly discovered that "Read" wasn't quite right. After spending no
less than 10 minutes unsuccessfully trying variations -- such as "Read-only" and
"read only" -- and searching the Web for the documented list of available
policies, I ended up telling my colleague to just "punt" and use the UI instead.

Revisiting the issue a few hours later, I just spent a couple of minutes
cranking out a console application to help me understand the available options
for the {{< kbd "permissionlevel" >}} parameter:

```C#
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

{{< console-block-start >}}

{{< sample-block >}}

Full Control\
Full Read\
Deny Write\
Deny All

{{< /sample-block >}}

{{< console-block-end >}}

Therefore the command that I should have suggested to my colleague is:

{{< console-block >}}

stsadm.exe -o addpermissionpolicy -url
[http://foobar/sites/Migration](http://foobar/sites/Migration) -userlogin "NT
AUTHORITY\Authenticated Users" -permissionlevel "Full Read"

{{< /console-block >}}
