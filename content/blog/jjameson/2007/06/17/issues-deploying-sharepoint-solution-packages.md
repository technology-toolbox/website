---
title: Issues Deploying SharePoint Solution Packages
date: 2007-06-17T07:44:00-06:00
excerpt:
  Several weeks ago, I converted our deployment process to use SharePoint
  solution packages instead of the batch scripts that we had been using in our
  Development environment. One of the issues that I discovered along the way is
  that SharePoint is rather...
aliases:
  [
    "/blog/jjameson/archive/2007/06/16/issues-deploying-sharepoint-solution-packages.aspx",
    "/blog/jjameson/archive/2007/06/17/issues-deploying-sharepoint-solution-packages.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/06/17/issues-deploying-sharepoint-solution-packages.aspx"
---

Several weeks ago, I converted our deployment process to use SharePoint solution
packages instead of the batch scripts that we had been using in our Development
environment. One of the issues that I discovered along the way is that
SharePoint is rather finicky when it comes to running the stsadm.exe command to
deploy your solution package.

I noted in a previous post that we have created a PublishingLayouts feature
containing our custom master pages, images, and stylesheets -- similar in
structure to the feature provided in Microsoft Office SharePoint Server (MOSS)
2007. Creating the WSP file was quite straightforward, as was adding it to the
solution store, using the following command:

```Console
stsadm -o addsolution -filename Fabrikam.Project1.PublishingLayouts.wsp
```

However, as soon as I tried to deploy the solution using the following command:

```Console
stsadm -o deploysolution -name Fabrikam.Project1.PublishingLayouts -url http://foobar/ -local
```

I encountered the following error:

{{< div-block "errorMessage" >}}

> This solution contains no resources scoped for a Web application and cannot be
> deployed to a particular Web application.

{{< /div-block >}}

I must have spent 30 minutes trying to figure out why this command did not work
(because it worked just fine for other features that I had converted to deploy
with WSPs). It turns out that I needed to omit the {{< kbd "url" >}} parameter:

```Console
stsadm -o deploysolution -name Fabrikam.Project1.PublishingLayouts -local
```

The reason why the PublishingLayouts solution would not deploy with the {{< kbd
"url" >}} parameter is because, unlike the other features, there was no assembly
generated for the PublishingLayouts (since it was pure content).

I also encountered the following error when trying to deploy our custom
Workflows feature:

{{< div-block "errorMessage" >}}

> Elements of type 'Workflow' are not supported at the 'WebApplication' scope.
> This feature could not be installed.

{{< /div-block >}}

I found that I had to omit the {{< kbd "url" >}} parameter for this solution as
well.

I then decided to try omitting the {{< kbd "url" >}} parameter when deploying
all of the other solutions. Without the {{< kbd "url" >}} parameter, I was able
to deploy 7 of our 9 features. The remaining two produced the following error:

{{< div-block "errorMessage" >}}

> This solution contains resources scoped for a Web application and must be
> deployed to one or more Web applications.

{{< /div-block >}}

For these two features, I _had_ to specify the {{< kbd "url" >}} parameter when
invoking stsadm.exe, because the manifest.xml file for the WSP specifies a
`<SafeControl>` element. When deploying these two solutions, SharePoint needs to
know which Web.config file to merge the `<SafeControl>` elements into, and
therefore the {{< kbd "url" >}} parameter must be specified.

The bottom line is that if your solution specifies elements (a.k.a. "resources")
that need to be merged into a Web.config file (i.e. "for a Web application")
then you _must_ specify the {{< kbd "url" >}} parameter. If your solution does
not have an assembly or if your solution contains workflows, then you _cannot_
specify the {{< kbd "url" >}} parameter.
