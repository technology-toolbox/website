---
title: '"Error Creating Control" when using Microsoft Office SharePoint Designer 2007'
date: 2007-03-22T05:57:00-06:00
description:
  'If, like me, you happen to encounter the following user experience when
  attempting to edit a master page... Figure 1: SharePoint Designer - "Error
  Creating Control" See full-sized image. ...then follow these steps to resolve
  the issue: ...'
aliases:
  [
    "/blog/jjameson/archive/2007/03/21/error-creating-control-when-using-microsoft-office-sharepoint-designer-2007.aspx",
    "/blog/jjameson/archive/2007/03/22/error-creating-control-when-using-microsoft-office-sharepoint-designer-2007.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/03/22/error-creating-control-when-using-microsoft-office-sharepoint-designer-2007.aspx"
---

If, like me, you happen to encounter the following user experience when
attempting to edit a master page...

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/SharePoint-Designer-Error-Creating-Control-600x305.png"
alt="SharePoint Designer - \"Error Creating Control\"" class="screenshot"
height="305" width="600"
caption="Figure 1: SharePoint Designer - \"Error Creating Control\"" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/SharePoint-Designer-Error-Creating-Control-756x384.png)

...then follow these steps to resolve the issue:

1. Start **Control Panel** and then open **Administrative Tools**.
1. In Windows Vista, right-click **Microsoft .NET Framework 2.0 Configuration**
   and click **Run as administrator**. If you are not using Windows Vista, then
   right-click and select **Run as...** and specify an account that is a member
   of the local Administrators group (because you don't normally login with an
   administrative account, right?). Enter the appropriate credentials to start
   the configuration console.
1. In the **.NET Framework 2.0 Configuration** console, in the tree on the left,
   expand **My Computer**, and click **Runtime Security Policy**.
1. In the **Code Access Security Policy** window on the right, in the **Tasks**
   section, click **Adjust Zone Security**.
1. In the **Security Adjustment Wizard**, ensure **Make changes to this
   computer** is selected and then click **Next**.
1. Select **Local Intranet**, in the **Choose the level of trust for assemblies
   from this zone** section, click and drag the slider to **Full Trust**, and
   then click **Next**.
1. Confirm the setting in the **Summary of changes** and then click **Finish**.

Restart SharePoint Designer and you should be "good-to-go."
