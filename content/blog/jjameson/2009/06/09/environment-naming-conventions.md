---
title: Environment Naming Conventions
date: 2009-06-09T18:42:00-06:00
excerpt:
  One of the challenges I see in organizations that I work with is the lack of
  naming conventions for various environments -- or sometimes naming conventions
  that provide little or no value. For about the last ten years, I've been a
  strong proponent...
aliases:
  [
    "/blog/jjameson/archive/2009/06/09/environment-naming-conventions.aspx",
  ]
categories: ["My System", "SharePoint", "Development", "Infrastructure"]
tags: ["My System", "Simplify", "MOSS 2007", "Core Development", "WSS v3", "SQL Server", "Infrastructure"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/06/09/environment-naming-conventions.aspx"
---

One of the challenges I see in organizations that I work with is the lack of
naming conventions for various environments -- or sometimes naming conventions
that provide little or no value.

For about the last ten years, I've been a strong proponent of a simple "-suffix"
naming convention that is not only very easy to learn (and remember!) but also
makes things incredibly obvious to all team members and stakeholders on a
project.

Most enterprise IT organizations typically have a naming convention for servers
that includes things like the Active Directory domain the server is a member of,
the role of the server, and usually some unique identifier to distinguish
multiple servers in a farm. Other "ingredients" such as the datacenter the
server is physically located in may also be included.

For example, my favorite fictitious manufacturing company, Fabrikam
Technologies, has Production servers named:

- FAB-DC-01 (the first domain controller for the FABRIKAM domain),
- FAB-DC-02,
- FAB-WEB-01 (the first Web server in a farm),
- FAB-WEB-02,
- FAB-SQL-01A (the first node in a SQL Server cluster, for which the cluster is
  named FAB-SQL-01),
- FAB-SQL-01B,
- etc.

This is a great start. As you can see from these examples, it's pretty easy to
tell what the role of each server is.

However, in any organization of reasonable size, we certainly need more than
just the Production environment (PROD). At a minimum, I recommend using a Test
environment (TEST) and a Development environment (DEV). Note that this is in
addition to individual developer environments -- which I typically refer to as
"local" evironments (LOCAL). Depending on how much parallel development is
planned -- and the corresponding release schedule -- you might also need a
Maintenance environment (MAINT). However, many organizations can function
effectively with just the DEV-TEST-PROD triad.

So this begs the question, what names should we use for DEV and TEST?

Here is what I recommend:

Monikers in the Development environment utilize a "-dev" suffix. For example,
the Web server in DEV corresponding to the FAB-WEB-01 server in PROD is named
FAB-WEB-01-DEV. Likewise, the SQL Server in the Development environment is named
FAB-SQL-01A-DEV. Note that typically Development environments do not have a
cluster, but I still recommend following the same naming convention and even
adding a DNS entry for the DEV "cluster" name (e.g. FAB-SQL-01-DEV). Test
environments often do have a SQL Server cluster configuration (which is a great
place to validate your failover configuration and load testing).

Host headers in the Development environment also follow the "-dev" naming
convention. For example, if the internal name of the Fabrikam Web site in
Production is [http://fabrikam](http://fabrikam/), then
[http://fabrikam-dev](http://fabrikam-dev/) is the corresponding URL used in the
Development environment.

In order to distinguish sites on the local development VMs, the "-local" suffix
is used. For example, [http://fabrikam-local](http://fabrikam-local/) is the URL
on an individual developer's VM corresponding to
[http://fabrikam](http://fabrikam/) in Production. The
%WINDIR%\System32\Drivers\etc\hosts file is then used to associate these URLs
with the loopback address (127.0.0.1).
[Be sure you are aware of the workaround in [KB 896861](http://support.microsoft.com/kb/896861)
when using the loopback address.]

Similarly, monikers and host headers in the Test environment utilize a "-test"
suffix (e.g. FAB-WEB-01-TEST, FAB-SQL-01A-TEST, and
[http://fabrikam-test](http://fabrikam-test/)).

Also note that organizations typically create separate service accounts within
Active Directory for the Development, Test, and Production environments (and if
you are not currently doing this, I strongly recommend it). For example, suppose
**FABRIKAM\svc-web** is the service account for the application pool running
[http://fabrikam](http://fabrikam/) in Production. Then we would expect the Web
site in DEV to be running as **FABRIKAM\svc-web-dev** and the site in TEST to be
running as **FABRIKAM\svc-web-test**.

This ensures that only the minimal number of people need to know the passwords
for a particular environment. For example, either the whole Development team --
or just a small subset of team leads -- needs to know the password for
**FABRIKAM\svc-web-dev**, but they certainly don't know the password for
**FABRIKAM\svc-web-test** (since that environment is managed by the Test team).
Likewise, the Test team had better not know the password for the service account
in Production (since -- at least hopefully -- that environment is owned by a
separate Release Management team).

It is also important to point out how this naming convention is applied to fully
qualified domain names (FQDNs). For example, suppose the external address of
[http://fabrikam](http://fabrikam/) is
[http://www.fabrikam.com](http://www.fabrikam.com/). What then should we use for
the corresponding FQDN in DEV, or should we even configure an FQDN for DEV? The
answer to that latter question is a resounding "Yes!" and the answer to the
former is [http://www-dev.fabrikam.com](http://www-dev.fabrikam.com/). Note that
we almost certainly won't to expose DEV on the Internet -- although we may
choose to expose TEST (
[http://www-test.fabrikam.com](http://www-test.fabrikam.com/))

The primary reason I recommend using these FQDNs (aside from the fact that we
are following the simple "-suffix" naming convention) is that browsers behave
differently between intranet names (e.g.
[http://fabrikam-dev](http://fabrikam-dev/)) and FQDNs (e.g.
[http://www-dev.fabrikam.com](http://www-dev.fabrikam.com/)). In other words,
when you are trying to simulate a problem on the external site, you should
ensure that you are simulating the same security "zone" in DEV, or even in your
local environment (e.g.
[http://www-local.fabrikam.com](http://www-local.fabrikam.com/)).

Lastly, I want to mention documentation. Whenever I sit down to write an
Installation Guide -- or similar documentation -- I typically only specify the
values for the Production environment, even though I fully expect the document
to also be used to install and configure the Test and Development environments.
I simply include a brief section at the beginning of the document describing the
naming convention and that it is the responsibility of the developer or tester
to modify the server names, host headers, and service accounts appropriately as
he or she is following along when configuring DEV or TEST.

Since we typically have "infrastructure models" (i.e. Visio diagrams showing the
physical architecture of DEV, TEST, and PROD), identifying servers and host
headers in various environments is really straightforward.
