---
title: "Eliminate MBSA Warnings Using Default Security Settings Policy"
date: 2009-10-21T03:57:00-06:00
excerpt:
  "Another Group Policy object that I use in the \"Jameson Datacenter\" (a.k.a.
  my home lab) is one that I created a couple of years ago in order to eliminate
  various warnings from the Microsoft Baseline Security Advisor (MBSA). 
   To automatically change..."
aliases:
  [
    "/blog/jjameson/archive/2009/10/20/eliminate-mbsa-warnings-using-default-security-settings-policy.aspx",
    "/blog/jjameson/archive/2009/10/21/eliminate-mbsa-warnings-using-default-security-settings-policy.aspx",
  ]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Simplify", "Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/21/eliminate-mbsa-warnings-using-default-security-settings-policy.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/21/eliminate-mbsa-warnings-using-default-security-settings-policy.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Another Group Policy object that I use in the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab) is one that I created a couple of years ago in order to eliminate
various warnings from the
[Microsoft Baseline Security Advisor](http://technet.microsoft.com/en-us/security/cc184924.aspx)
(MBSA).

To automatically change the default security settings in the "Jameson
Datacenter", I defined a Group Policy (named **Default Security Settings
Policy**) with the following settings:

- **Computer Configuration**
  - **Policies**
    - **Windows Settings**
      - **Security Settings**
        - **Account Policies**
          - **Password Policy**
            - **Maximum password age: 60 days**
            - **Minimum password age: 1 day**
            - **Minimum password length: 8 characters**
        - **Local Policies**
          - **Security Options**
            - **Network security: LAN Manager authentication level: Send NTLMv2
              response only. Refuse LM & NTLM**
        - **System Services**
          - **TlntSvr**
            - **Startup Mode: Disabled**

I don't know about you, but I haven't used Telnet in almost fifteen years --
back when I used to work on Unix systems ;-)

This Group Policy is linked to the entire domain (i.e.
**corp.technologytoolbox.com**).

Note that I still use the **Default Domain Controllers Policy** to configure the
security settings on the domain controllers (and thus domain accounts). In other
words, the settings noted above only affect local accounts (e.g.
COLOSSUS\Administrator, not TECHTOOLBOX\jjameson).
