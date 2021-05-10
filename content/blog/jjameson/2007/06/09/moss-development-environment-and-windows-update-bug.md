---
title: MOSS Development Environment and a Windows Update Bug
date: 2007-06-09T06:38:00-06:00
description:
  In my previous post , I talked about splitting our Microsoft Office SharePoint
  Server (MOSS) 2007 Development environment (DEV) into multiple VMs. What I did
  not mention, however, is the nasty bug in Windows Update that I encountered
  along the way. ...
aliases:
  [
    "/blog/jjameson/archive/2007/06/08/moss-development-environment-and-windows-update-bug.aspx",
    "/blog/jjameson/archive/2007/06/09/moss-development-environment-and-windows-update-bug.aspx",
  ]
categories: ["SharePoint", "Infrastructure"]
tags: ["MOSS 2007", "ISA Server"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/06/09/moss-development-environment-and-windows-update-bug.aspx"
---

In my [previous post](/blog/jjameson/2007/06/09/virtual-server-issues), I talked
about splitting our Microsoft Office SharePoint Server (MOSS) 2007 Development
environment (DEV) into multiple VMs. What I did not mention, however, is the
nasty bug in Windows Update that I encountered along the way.

Before I can explain the bug I first need to describe the environment. The
following model shows the important pieces of the environment.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/MOSS-2007-Development-Environment-600x456.jpg"
alt="MOSS 2007 development environment" height="456" width="600"
title="Figure 1: MOSS 2007 development environment" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/MOSS-2007-Development-Environment-785x596.jpg)

DEV is comprised of five VMs running on a single Virtual Server 2005 host:

- DC1 - Active Directory domain controller for our development domain
- SQL1 - SQL Server
- SSP1 - MOSS 2007 Shared Services Provider (hosts Central Administration and
  indexes content)
- WEB1 - MOSS 2007 Web server (hosts various Web applications and provides
  search functionality)
- ISA1 - ISA Server

ISA Server is used to securely publish the Web applications (in other words, it
allows us to access DEV without having to TermServ into the Virtual Server host,
launch VMRC, and browse to a site). It also acts as the gateway for the DEV VMs
so that they can access other (physical) servers outside DEV (for example, to
copy files to or from another server).

Using the Virtual Server Administration Website, I created virtual networks
(VLAN1 and VLAN2) to represent the "front-end" and "backend" networks for the
MOSS servers.

- VLAN1 is used internally within DEV for all infrastructure traffic (such as
  Kerberos authentication and database queries).
- VLAN1 is also used to allow the various VMs to connect to the "outside world"
  (the default gateway on the various DEV VMs is set to the IP address of ISA1
  on VLAN1; therefore ISA1 acts as the router between the physical network and
  the virtual network used for the Development environment).
- HTTP requests to DEV are routed through ISA1 on VLAN2 to WEB1 (for most Web
  applications) or SSP1 (for Central Administration and the SSP administration
  site).

Also note that in my current customer's environment, you must use a proxy server
(PROXY1) in order to access the Internet. Note that PROXY1 is not an ISA Server
so I am not using the "firewall chaining" feature in ISA Server. This customer
does not use WPAD so you have to manually configure the proxy server and specify
port 8088. This is where I discovered the bug in Windows Update...

Since I wanted to allow the DEV VMs to connect to the Internet, I configured the
proxy server on each VM (to point to PROXY1) and added a rule on ISA1 to allow
outbound HTTP traffic from the internal network (i.e. VLAN1 and VLAN2) to the
external network. However, I quickly discovered that this did not work because,
as I mentioned earlier, the proxy is configured for port 8088. Therefore I added
a new custom protocol to the access rule to allow HTTP on port 8088. Bingo! I
could now access the Internet from each VM in DEV.

However, when I browsed to Windows Update, I discovered that while it was able
to determine what updates I needed, it was not able to actually download and
install the patches. Looking at the logs on ISA1, I noticed many requests on
port 80 after approving the installation of the patches. After researching the
problem a little, several support incidents indicated that the proxycfg utility
needed to be used to get BITS to work with a proxy server. (BITS is used by
Windows Update to download -- but not detect -- the patches).

I ran "proxycfg -u" to import my settings from Internet Explorer. However, I
still was not able to download the patches. I rebooted (hoping that the BITS
would start using the proxy settings). No luck. I even found one SOX article --
which is basically an FAQ created internally based on common SRXs (service
requests, a.k.a. support incidents) -- that stated that you need to start a
command prompt as LocalSystem (the author creatively recommended using a
one-time scheduled task to do this) and running proxycfg there. Well, that may
have solved some people's problems but it did not solve mine.

What I discovered is that when ISA1 allows Windows Update to pass requests to
PROXY1, then that proxy server receives the request and subsequently returns an
error code (since it requires port 8088 to be used). When the error is received
by Windows Update, it tries again with a different destination IP (hoping that
it can somehow find a Windows Update server). This process repeats for a couple
of iterations and then it finally times out (meaning you don't get any patches).
Apparently the error returned from PROXY1 when a request comes in on port 80 is
not acknowledged by the BITS client as being a "real" problem, so BITS goes
ahead and tries again (until it eventually gives up).

The solution that I discovered is that I needed to remove HTTP (port 80) from
the access rule on ISA1 (but still allowing port 8088). This way, when Windows
Update attempts to download the patches, it receives an "access denied" error
from ISA1. The underlying BITS client then proceeds to use the proxy server
configuration (i.e. it requests the download on port 8088).

Way more infrastructure than I care for, but definitely an educational
experience for me.
