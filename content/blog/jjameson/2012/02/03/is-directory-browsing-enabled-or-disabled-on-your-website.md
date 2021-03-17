---
title: "Is directory browsing enabled or disabled on your website?"
date: 2012-02-03T07:57:15-07:00
excerpt:
  "Starting with IIS 6, directory browsing is disabled by default -- but are you
  really sure this is the way your website is configured?"
aliases: ["/blog/jjameson/archive/2012/02/03/is-directory-browsing-enabled-or-disabled-on-your-website.aspx"]
draft: true
categories: ["Development"]
tags: ["Web Development"]
---

A few weeks ago I was investigating an issue with the Technology Toolbox website
and I discovered that directory browsing was enabled. This came as quite a shock
since I was 99% certain that directory browsing is *disabled* by default in IIS.

If I recall correctly, a big part of Windows Server 2003 (and IIS 6) was to
change the default settings to provide a minimal attack surface. [Remember Code
Red, Nimbda, et al.? Boy, I definitely don't miss those days *at all* --
especially since I was an employee of Microsoft throughout those times. Talk
about developing a "tough skin" ;-)]

After a quick search for "WinHost directory browsing" I quickly discovered this
is actually documented on the WinHost site:

{{< blockquote "font-italic" >}}

... By default, directory browsing is enabled.

{{< /blockquote >}}

{{< reference
title="WinHost Support Portal - Can I control directory browsing on my site?"
linkHref="http://support.winhost.com/KB/a590/can-i-control-directory-browsing-on-my-site.aspx" >}}

Well shiver me timbers, mateys!

Using IIS Manager, I made a quick change to disable directory browsing on
TechnologyToolbox.com. This added the following element to the Web.config file:

```
<configuration>
  ...
  <system.webServer>
    ...
    <directoryBrowse enabled="false" />
  </system.webServer>
</configuration>
```

The next time I create a new ASP.NET application to be hosted externally, I
think it would be wise to just add this to the Web.config file in the beginning.
