---
title: SQL Server Authentication Modes
date: 2007-03-23T07:12:00-06:00
excerpt:
  A fellow consultant here in Denver sent out a message yesterday inquiring
  about formal recommendations regarding the use of Windows Authentication vs.
  SQL Authentication. It seems that quite a few customers out there ponder these
  choices for a variety...
aliases:
  [
    "/blog/jjameson/archive/2007/03/22/sql-server-authentication-modes.aspx",
    "/blog/jjameson/archive/2007/03/23/sql-server-authentication-modes.aspx",
  ]
draft: true
categories: [""]
tags: ["SQL Server"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/03/23/sql-server-authentication-modes.aspx"
---

A fellow consultant here in Denver sent out a message yesterday inquiring about
formal recommendations regarding the use of Windows Authentication vs. SQL
Authentication. It seems that quite a few customers out there ponder these
choices for a variety of reasons.

I dug up an email that I had sent to a different customer (that, for whatever
reason, chooses to use SQL Authentication over Windows Authentication):

***

**From:** Jeremy Jameson\
**To:** [names removed to protect the innocent]\
**Subject:** SQL Server Authentication Modes

Here is some info from MSDN:

> **SQL Server Authentication Modes**
>
> Microsoft® SQL Server™ can operate in one of two security (authentication)
> modes:
>
> - Windows Authentication Mode (Windows Authentication)\
>   Windows Authentication mode allows a user to connect through a Microsoft
>   Windows NT® 4.0 or Windows® 2000 user account.
> - Mixed Mode (Windows Authentication and SQL Server Authentication)
> - **Security Note** When possible, use Windows Authentication.
>
> ...
>
> Windows Authentication has certain benefits over SQL Server Authentication,
> primarily due to its integration with the Windows NT 4.0 and Windows 2000
> security system. Windows NT 4.0 and Windows 2000 security provides more
> features, such as secure validation and encryption of passwords, auditing,
> password expiration, minimum password length, and account lockout after
> multiple invalid login requests.
>
> ...
>
> SQL Server Authentication is provided for backward compatibility because
> applications written for SQL Server version 7.0 or earlier may require the use
> of SQL Server logins and passwords. [...]
>
> ...
>
> Even though Windows Authentication is recommended, SQL Server Authentication
> may be required for connections with clients other than Windows NT 4.0 and
> Windows 2000 clients; it may also be necessary for legacy applications.
>
> Pasted from
> &lt;[http://msdn.microsoft.com/library/en-us/adminsql/ad\_security\_47u6.asp?frame=true](http://msdn.microsoft.com/library/en-us/adminsql/ad_security_47u6.asp?frame=true)&gt;
>
> Windows authentication is more secure than SQL authentication for the
> following reasons:
>
> - Credentials are managed for you and the credentials are not transmitted over
>   the network.
> - You avoid embedding user names and passwords in connection strings.
> - Logon security improves through password expiration periods, minimum
>   lengths, and account lockout after multiple invalid logon requests. This
>   mitigates the threat from dictionary attacks.
>
> Pasted from
> &lt;[http://msdn.microsoft.com/practices/compcat/default.aspx?pull=/library/en-us/dnnetsec/html/SecNetch12.asp](http://msdn.microsoft.com/practices/compcat/default.aspx?pull=/library/en-us/dnnetsec/html/SecNetch12.asp)&gt;
>
> I remember speaking to a SQL Server Program Manager a few years ago at a
> session where he stated that as of SQL Server [2000] SP3 (and some specific
> version of MDAC which I don't recall) the authentication between the client
> and SQL Server is automatically secured using SSL - assuming a trusted
> certificate is installed on the SQL Server. Perhaps that is the route that is
> being used here at Fabrikam [ed. actual customer name substituted]. In this
> case, the argument of sending credentials in clear text no longer applies.
> However, the storage of username/password in various config files is still
> valid regardless. Encrypting the credentials in the config files vastly
> improves the situation, but this will never be as secure as using Windows
> Authentication (primarily due to the increased security provided by standard
> protocols such as Kerberos).

***

Note that some improvements have been made in SQL Server 2005 (such as password
expiration policies) that mitigate some of the items I mentioned in my original
email. Nevertheless, if you have an Active Directory infrastructure in place --
which any organization over 2 people should ;-) -- and you are connecting from
another server that you can get a valid Kerberos ticket on (i.e. in the same
domain or in a trusted domain) then you should use Windows Authentication when
connecting to SQL Server.

If you have the SQL Server in a separate domain (perhaps inside the corporate
network but used by Web servers in the DMZ) then is it worth the effort to
establish a cross-forest trust in order to be able to use Windows
Authentication?

IMO, absolutely!
