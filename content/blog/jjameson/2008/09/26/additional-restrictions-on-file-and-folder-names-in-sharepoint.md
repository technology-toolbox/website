---
title: Additional Restrictions on File and Folder Names in SharePoint
date: 2008-09-26T11:03:00-06:00
excerpt:
  Up until this morning, I thought I knew what the restrictions were for file
  and folder names in Windows SharePoint Services 3.0 (WSS v3) and Microsoft
  Office SharePoint Server 2007 (MOSS 2007). However, I learned the hard way
  (i.e. during the process...
aliases:
  [
    "/blog/jjameson/archive/2008/09/25/additional-restrictions-on-file-and-folder-names-in-sharepoint.aspx",
    "/blog/jjameson/archive/2008/09/26/additional-restrictions-on-file-and-folder-names-in-sharepoint.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2008/09/26/additional-restrictions-on-file-and-folder-names-in-sharepoint.aspx"
---

Up until this morning, I thought I knew what the restrictions were for file and
folder names in Windows SharePoint Services 3.0 (WSS v3) and Microsoft Office
SharePoint Server 2007 (MOSS 2007). However, I learned the hard way (i.e. during
the process of migrating content to the Production environment) that there are
some additional restrictions that are not currently listed in the following KB
article:

{{< reference
title="Information about the characters that you cannot use in sites, folders, and files in SharePoint Portal Server 2003 or in SharePoint Server 2007"
linkHref="http://support.microsoft.com/default.aspx?scid=kb;en-us;905231" >}}

It turns out that the restrictions noted in this KB article are incomplete.

While bulk importing content from the file system (e.g. images) via the PRIME
API, I found out that SharePoint does not allow folders that end in ".files" or
"\_files" (which my customer just happens to have). This is apparently a
[known issue](http://technet.microsoft.com/en-us/library/cc261812.aspx) (at
least as early as 2008-06-28).

I have initiated a change request to correct KB 905231, as follows.

In addition, file and folder names cannot end with:

- .files
- \_files
- -Dateien
- \_fichiers
- \_bestanden
- \_file
- \_archivos
- -filer
- \_tiedostot
- \_pliki
- \_soubory
- \_elemei
- \_ficheiros
- \_arquivos
- \_dosyalar
- \_datoteke
- \_fitxers
- \_failid
- \_fails
- \_bylos
- \_fajlovi
- \_fitxategiak

I'll update this post after my change request has been reviewed.
