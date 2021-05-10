---
title: Updated Path to tf.exe for TFS 2010 Builds
date: 2010-05-05T04:32:00-06:00
description:
  After upgrading my Team Foundation Server (TFS) 2008 environment to TFS 2010 ,
  I found that I needed to tweak my TfsBuild.proj file in order to successfully
  build on my new TFS 2010 build server. In a previous post, I detailed the
  process that I recommend...
aliases:
  [
    "/blog/jjameson/archive/2010/05/04/updated-path-to-tf-exe-for-tfs-2010-builds.aspx",
    "/blog/jjameson/archive/2010/05/05/updated-path-to-tf-exe-for-tfs-2010-builds.aspx",
  ]
categories: ["My System", "Development"]
tags: ["My System", "Core Development", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/05/05/updated-path-to-tf-exe-for-tfs-2010-builds.aspx"
---

After
[upgrading my Team Foundation Server (TFS) 2008 environment to TFS 2010](/blog/jjameson/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010-overview),
I found that I needed to tweak my TfsBuild.proj file in order to successfully
build on my new TFS 2010 build server.

In a previous post, I detailed the process that I recommend for
[incrementing the assembly version with each build](/blog/jjameson/2010/03/25/incrementing-the-assembly-version-for-each-build),
in which I explained how to add a property so that we can use the TFS
command-line utility to checkout the assembly version files and subsequently
check them back in:

```XML
<PropertyGroup>
  <TeamFoundationVersionControlTool>&quot;$(TeamBuildRefPath)\..\tf.exe&quot;</TeamFoundationVersionControlTool>
</PropertyGroup>
```

While this worked great on a TFS 2008 build server, the path to the TFS
command-line utility has changed for a TFS 2010 build server.

To use the same technique on a TFS 2010 build server, specify the following
instead:

```XML
<PropertyGroup>
  <TeamFoundationVersionControlTool>&quot;$(VS100COMNTOOLS)..\IDE\tf.exe&quot;</TeamFoundationVersionControlTool>
</PropertyGroup>
```
