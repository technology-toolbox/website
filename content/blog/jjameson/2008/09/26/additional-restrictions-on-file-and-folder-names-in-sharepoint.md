---
title: "Additional Restrictions on File and Folder Names in SharePoint"
date: 2008-09-26T05:03:00-07:00
excerpt: "Up until this morning, I thought I knew what the restrictions were for file and folder names in Windows SharePoint Services 3.0 (WSS v3) and Microsoft Office SharePoint Server 2007 (MOSS 2007). However, I learned the hard way (i.e. during the process..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/09/26/additional-restrictions-on-file-and-folder-names-in-sharepoint.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/09/26/additional-restrictions-on-file-and-folder-names-in-sharepoint.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

Up until this morning, I thought I knew what the restrictions were for file
and folder names in Windows SharePoint Services 3.0 (WSS v3) and Microsoft Office
SharePoint Server 2007 (MOSS 2007). However, I learned the hard way (i.e. during
the process of migrating content to the Production environment) that there are
some additional restrictions that are not currently listed in the following
KB article:

<cite>Information about the characters that you cannot use in sites, folders,
and files in SharePoint Portal Server 2003 or in SharePoint Server 2007</cite>
[http://support.microsoft.com/default.aspx?scid=kb;en-us;905231](http://support.microsoft.com/default.aspx?scid=kb;en-us;905231)

It turns out that the restrictions noted in this KB article are incomplete.

While bulk importing content from the file system (e.g. images) via the PRIME
API, I found out that SharePoint does not allow folders that end in ".files"
or "\_files" (which my customer just happens to have). This is apparently a
[known issue](http://technet.microsoft.com/en-us/library/cc261812.aspx)
(at least as early as 2008-06-28).

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
