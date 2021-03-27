---
title: Configure IntelliMirror Using Group Policy
date: 2009-10-21T04:57:00-06:00
excerpt:
  "Yet another Group Policy object that I use in the \"Jameson Datacenter\"
  (a.k.a. my home lab) is one to automatically configure roaming profiles and
  redirect the Desktop and Documents folders to a server(a.k.a.
  \"IntelliMirror\"). Even though I don't have..."
aliases:
  [
    "/blog/jjameson/archive/2009/10/20/configure-intellimirror-using-group-policy.aspx",
    "/blog/jjameson/archive/2009/10/21/configure-intellimirror-using-group-policy.aspx",
  ]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Simplify", "Windows Server", "Infrastructure"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/10/21/configure-intellimirror-using-group-policy.aspx"
---

Yet another Group Policy object that I use in the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a.
my home lab) is one to automatically configure roaming profiles and redirect the
Desktop and Documents folders to a server(a.k.a. "IntelliMirror").

Even though I don't have many users in my Active Directory domain -- it's not
like I have eight kids, just one -- I still want to keep user data centrally
managed on a server that I backup regularly. Besides, I find it really
frustrating to have some items on your desktop on one computer, but a different
set of desktop items on another computer (or VM).

To automatically configure this in the "Jameson Datacenter", I defined a Group
Policy (named **Default User Data and Settings Policy**) with the following
settings:

- **User Configuration**
  - **Policies**
    - **Windows Settings**
      - **Folder Redirection**
        - **AppData(Roaming)**
          - **Setting: Basic (Redirect everyone's folder to the same location)**
            - Path: \\\\beast\Users$\\%USERNAME%\Application Data
          - **Options**
            - Grant user exclusive rights to AppData(Roaming): Enabled
            - Move the contents of AppData(Roaming) to the new location: Enabled
            - Also apply redirection policy to Windows 2000, Windows 2000
              server, Windows XP, and Windows Server 2003 operating systems:
              Enabled
            - Policy Removal Behavior: Leave contents
        - **Desktop**
          - **Setting: Basic (Redirect everyone's folder to the same location)**
            - Path: \\\\beast\Users$\\%USERNAME%\Desktop
          - **Options**
            - Grant user exclusive rights to Desktop: Enabled
            - Move the contents of Desktop to the new location: Enabled
            - Also apply redirection policy to Windows 2000, Windows 2000
              server, Windows XP, and Windows Server 2003 operating systems:
              Enabled
            - Policy Removal Behavior: Leave contents
        - **Documents**
          - **Setting: Basic (Redirect everyone's folder to the same location)**
            - Path: \\\\beast\Users$\\%USERNAME%\Documents
          - **Options**
            - Grant user exclusive rights to Documents: Enabled
            - Move the contents of Documentsto the new location: Enabled
            - Also apply redirection policy to Windows 2000, Windows 2000
              server, Windows XP, and Windows Server 2003 operating systems:
              Enabled
            - Policy Removal Behavior: Leave contents
        - **Music**
          - Setting: Follow the Documents folder
        - **Pictures**
          - Setting: Follow the Documents folder
        - **Videos**
          - Setting: Follow the Documents folder

{{< div-block "note" >}}

> **Note**
>
> Those of you that have a very keen eye (and also a photographic memory) might
> recall that in a previous post, I listed BEAST as a database server (it is
> currently running SQL Server 2005). Yes, it's true, I'm breaking one of my own
> cardinal sins by having a SQL Server also serve as a file server. I don't
> recommend doing this unless, like me, you are trying to go as cheap as
> possible -- and, even then, only for a lab environment like mine.

{{< /div-block >}}

In order to allow users access to create their own folders on \\\\BEAST\Users$,
I have configured the following permissions on C:\BackedUp\Users:

- Domain Users
  - Apply onto: This folder only
  - Permissions
    - List Folder / Read Data
    - Create Folders / Append Data
- CREATOR OWNER
  - Apply onto: Subfolders and files only
  - Permissions
    - Full Control

I also created a hidden share for the C:\BackedUp\Users folder and granted
**Full Control** to **Authenticated Users** (since the NTFS permissions above
ultimately determine the level of access for all users).

Thus when a new user logs in for the first time, a corresponding folder is
created on the server and all of the user's files are stored on the server.
