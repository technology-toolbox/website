---
title: "Enabling Remote Desktop via Group Policy"
date: 2009-10-13T22:53:00-07:00
excerpt: "In a previous post, I provided some details on the \"Jameson Datacenter\" (a.k.a. my home lab). In a follow-up post, I also discussed the Active Directory domain structure and mentioned how I use the Group Policy feature of Active Directory to \"effortlessly..."
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Windows Server", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/14/enabling-remote-desktop-via-group-policy.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/14/enabling-remote-desktop-via-group-policy.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In a previous post, I provided some details on the ["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a. my home lab). In a follow-up post, I also discussed the [Active Directory domain structure](/blog/jjameson/2009/10/02/active-directory-domain-structure-in-the-jameson-datacenter) and mentioned how I use the Group Policy feature of Active Directory to "effortlessly" configure new servers.

For example, I have defined a Group Policy (named **Enable Terminal Services Policy**) with the following settings:

- **Computer Configuration**
  - **Policies**
    - **Windows Settings**
      - **Security Settings**
        - **Windows Firewall with Advanced Security**
          - **Inbound Rules**
            - **Remote Desktop (TCP-In)**
              - **Enabled: Yes**
              - **Action: Allow**
    - **Administrative Templates**
      - **Windows Components**
        - **Terminal Services**
          - **Terminal Server**
            - **Connections**
              - **Allow users to connect remotely using Terminal Services: Enabled**

By linking this Group Policy to the appropriate OUs (e.g. **Development/Resources/Servers**) I do not have to manually enable Remote Desktop connections on each new server (e.g. a new SharePoint development VM). Instead this is automatically configured as soon as I join a server to the domain and reboot.

I'll cover some of the other Group Policy objects in subsequent posts.

