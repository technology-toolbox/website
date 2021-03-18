---
title: "Be \"In the Zone\" to Avoid Entering Credentials"
date: 2007-03-22T04:50:00-06:00
excerpt:
  Since I tend to work on server products such as Microsoft Office SharePoint
  Server 2007, I frequently see developers and other team members that I work
  with constantly entering their credentials when browsing to some Web page on a
  server. While this experience...
aliases:
  [
    "/blog/jjameson/archive/2007/03/21/be-in-the-zone-to-avoid-entering-credentials.aspx",
    "/blog/jjameson/archive/2007/03/22/be-in-the-zone-to-avoid-entering-credentials.aspx",
  ]
draft: true
categories: ["My System"]
tags: ["My System"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/03/22/be-in-the-zone-to-avoid-entering-credentials.aspx"
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2007/03/22/be-in-the-zone-to-avoid-entering-credentials.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/03/22/be-in-the-zone-to-avoid-entering-credentials.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Since I tend to work on server products such as Microsoft Office SharePoint
Server 2007, I frequently see developers and other team members that I work with
constantly entering their credentials when browsing to some Web page on a
server. While this experience might be considered useful in some scenarios (e.g.
testing the site as a different user), it is painful to watch people enter the
same username and password that they are already specified when logging into
their laptop, desktop, or VM.

Personally speaking, I loathe having to enter credentials to authenticate
against any Web application, but especially one that is on the local intranet,
or worse, one that is running locally on a VM that I happen to be using for
development purposes.

It seems that many people are not aware of the fact that in order to provide a
more secure out-of-the-box configuration, Internet Explorer will not
automatically attempt to authenticate you to a site unless you explicitly tell
it to do so. In other words, you tell IE, "go ahead and transparently
authenticate me when I browse to [http://foobar](http://foobar/), because a) I
trust [http://foobar](http://foobar/), b) I know that
[http://foobar](http://foobar/) supports integrated Windows authentication (i.e.
Kerberos or NTLM), and c) I am just going to enter the same username and
password that I am already logged in with, so entering these again would just be
a waste of my time."

The default settings are set to authenticate automatically only to those servers
in the "Local intranet zone" -- which, for Windows Server 2003, only contains
sites like [http://localhost](http://localhost/) and
[https://localhost](https://localhost/). In other words, to provide the most
secure out-of-the-box configuration, IE will not attempt to authenticate
automatically to any arbitrary intranet URL (i.e. no dots) -- you need to
explicitly tell IE that it is okay to do so. Fortunately, this is very easy to
configure.

To add a Web site to the Local intranet zone and configure IE to automatically
authenticate:

1. In Internet Explorer, on the **Tools** menu, click **Internet Options**.
2. On the **Security** tab, in the **Select a zone to view or change security
   settings** box, click **Local intranet**, and then click **Sites**.
3. At this point, depending on whether you are using Windows Vista or Windows
   Server 2003, you may need to click the **Advanced** button to view the list
   of sites in this zone.
4. Clear the **Require server verification (https:) for all sites in this zone**
   check box.
5. In the **Add this Web site to the zone** box, type the URL for the Web site
   (e.g. [http://foobar](http://foobar/)), and then click **Add**.
6. Click **Close** to close the **Local intranet** dialog box.
7. In the **Security level for this zone** section, click **Default level**
   (this will ensure that the default option to authenticate automatically only
   to URLs in the Local intranet zone is enabled ).
8. Click **OK** to close the **Internet Options** dialog box.

So, the next time you see that generic login box pop up when browsing to a local
or intranet site, rather than entering the same username and password that you
are currently authenticated with, click **Cancel** and then follow the steps
above instead.
