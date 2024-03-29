---
title: Deleting Large Sites and Recovering from the Failed Deletion of a Site
date: 2007-05-07T06:28:00-06:00
description:
  Apparently you cannot delete a site containing a large amount of content in
  Microsoft Office SharePoint Server (MOSS) 2007. Last week I deleted -- or
  rather, attempted to delete – a Document Center site in our Test environment
  in order to recreate it...
aliases:
  [
    "/blog/jjameson/archive/2007/05/06/deleting-large-sites-and-recovering-from-the-failed-deletion-of-a-site.aspx",
    "/blog/jjameson/archive/2007/05/07/deleting-large-sites-and-recovering-from-the-failed-deletion-of-a-site.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/05/07/deleting-large-sites-and-recovering-from-the-failed-deletion-of-a-site.aspx"
---

Apparently you cannot delete a site containing a large amount of content in
Microsoft Office SharePoint Server (MOSS) 2007. Last week I deleted -- or
rather, attempted to delete -- a Document Center site in our Test environment in
order to recreate it (to fix some bugs in our document migration process).
Unfortunately, the deleteweb.aspx page timed out and left the database in an
inconsistent state -- attempts to browse to the site
([http://server/Archive/Library](http://server/Archive/Library)) returned HTTP
400 - Bad Request. The "deleted" site showed up in the list of site collections
in Central Administration (although you could not select it in order to delete
it) but it did not show up in "{{< kbd "stsadm -o enumsites" >}}". Attempts to
recreate the site (upon activation of one of our custom features) also failed
with the following error:

{{< div-block "errorMessage" >}}

> The system cannot find the path specified. (Exception from HRESULT:
> 0x80070003)

{{< /div-block >}}

Something similar happened last month on my initial deployment to TEST (I
pressed {{< kbd "CTRL+C" >}} while activating one of the features that creates
sites and ended up corrupting the database). At that time I simply deleted the
Web application and recreated it.

We are planning on rebuilding our Test environment at least once sometime during
the next couple of weeks. However, I did not want to perform a rebuild last week
since it would interrupt the Test team (i.e. the /Archive/Library site is only a
fraction of the feature set we were currently testing and we have several
hundred gigabytes of content in other site collections to support the testing of
those features). I was prepared to delete the entire Web application but only as
a last resort.

Since my goal was to get TEST up and running as quickly as possible, I ended up
using SQL Profiler while browsing the Central Administration page that listed my
defunct site. This showed me that the site was still listed in the SiteMap table
of the SharePoint_Config database. I then ran the following statement to remove
the site:

```SQL
DELETE FROM SiteMap
WHERE Id = 'A4E86B02-49E1-4960-843E-125BB13D6C54'
```

This allowed me to activate the feature again which recreated the
/Archive/Library site. It is important to note that _executing SQL statements
directly against any SharePoint database is completely unsupported_ (meaning you
shouldn't bother calling PSS once you have directly made any updates to the
database). However, in our case, this risk was mitigated by the fact that we'll
be deleting the databases during a rebuild soon. Otherwise, I definitely would
have opted to open a case with PSS and let them guide us through the resolution
of the issue.

So...lessons learned:

- Do not use the Web application to delete a large site (e.g. a Document Center
  with lots of content) since it could timeout and leave your SharePoint
  databases in an inconsistent state; use "{{< kbd
  "stsadm.exe -o deletesite" >}}" or "{{< kbd "stsadm.exe -o deleteweb" >}}"
  instead
- Not all data for a particular Web application is stored in the content
  database (e.g. WSS_Content); a small amount of data about the site is also
  stored in SharePoint_Config
- Deleting a site does not perform a distributed transaction across the content
  database and the configuration database (the root of the issue when deleting a
  large site)
