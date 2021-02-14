---
title: "Shared files and assemblies in ASP.NET applications (a.k.a. Building TechnologyToolbox.com, part 17)"
date: 2012-01-26T00:36:03+08:00
excerpt: "This post describes a couple of scenarios where you might need to share files and assemblies within an ASP.NET website, as well as some tricks for making this completely painless (from a development and deployment perspective)."
draft: true
categories: ["Development", "My System"]
tags: ["Subtext", "TFS", "Visual Studio", "Web Development"]
---

Okay, so I've recommended
[using ELMAH for error handling](/blog/jjameson/2012/01/22/building-technologytoolbox-com-part-14) in your ASP.NET Web applications and also
showed you a
[CAPTCHA control](/blog/jjameson/2012/01/25/building-technologytoolbox-com-part-16) that you can use with the Subtext blog engine as well as
plain ol' ASP.NET apps.

Now, suppose you want to use ELMAH and/or the CAPTCHA control in both an
ASP.NET Web application and a Subtext-powered blog *at the same time*.
In other words, suppose you have a custom ASP.NET application that renders part
of a website (e.g. "/" and "/Services") and uses Subtext to serve up other parts
(e.g. URLs under "/blog").

For example, I've
[shown before](/blog/jjameson/2011/10/18/introducing-technologytoolbox-com) how the Technology Toolbox website is actually built from two
distinct Visual Studio solutions (specifically the "Caelum" solution in Visual
Studio 2010 and the "Subtext" solution in Visual Studio 2008).

![Solution architecture for TechnologyToolbox.com](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Solution-Architecture.jpg)

    	Figure 1: Solution architecture for TechnologyToolbox.com

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Solution-Architecture.jpg)

Note that the Caelum website is a "classic" ASP.NET Web application, while
Subtext is an ASP.NET MVC application. During the deployment process, the two
solutions are merged together.

The **blog** folder is configured as a separate application
in IIS. It contains the Subtext solution and a few updated/additional files
from the "Caelum" solution -- such as the site map file and the custom blog
skin.

Due to the ELMAH configuration specified in the root Web.config file and
custom controls specified in the Subtext skin created for Technology Toolbox,
there are actually dependencies between the **blog** application
and the Caelum solution.

Consequently, it is necessary to copy files from the Caelum solution into
the corresponding locations within the Subtext **blog** application.

For example, to use the custom CAPTCHA control described in my previous post,
the Subtext skin specifies the following (in PostComment.ascx):

```
<%@ Control Language="C#" EnableTheming="false" AutoEventWireup="false"
  Inherits="Subtext.Web.UI.Controls.PostComment" %>
<%@ Register src="~/Controls/Captcha/Captcha.ascx" TagName="Captcha"
  TagPrefix="uc1" %>
...
<div id="commentForm">
    <h3>Add Comment</h3>
    ...
    <uc1:Captcha runat="server" ValidationGroup="SubtextComment" />
    ...
</div>
```

```
...
```

At first glance, you might think `"~/Controls/Captcha/Captcha.ascx"`
refers to the user control in the **www.technologytoolbox.com\Controls\Captcha**
folder, but it actually refers to **blog\Controls\Captcha\Captcha.ascx**
(because the **blog** folder is configured as a separate application
in IIS).

Similarly, when the Subtext application goes looking for the Caelum assemblies
(e.g. for the base class of the user control -- in other words, the code-behind),
it only looks in the **blog\bin** folder (not the **www.technologytoolbox\bin**
folder). The fact that ASP.NET does not probe for assemblies in the root
**bin** folder is very important -- especially when it comes to
using ELMAH.

If ELMAH is configured in the Web.config file of the root application (i.e.
**www.technologytoolbox\Web.config**) but the ELMAH assembly is
not in the **blog\bin** folder (and presumably not in the GAC either),
then any request processed by Subtext will generate an unhandled exception:

> Exception type: ConfigurationErrorsException
>
> Exception message: Could not load file or assembly 'Elmah' or one of its
> dependencies. The system cannot find the file specified.

Copying the ELMAH assembly from the root **bin** folder (where
it was deployed as a result of adding it as a reference to the **Website** project in the Caelum solution) into the **blog\bin**
folder resolves the issue. Similarly, the CAPTCHA user control (Captcha.ascx)
and "Caelum" assemblies need to be copied as well.

However, while manually copying files certainly works, It would quickly become
tedious (while the ELMAH assembly won't change very often, the Caelum assemblies
would need to be "refreshed" after each build and deployment). Fortunately,
there's actually a much better approach to "sharing" files and assemblies in
ASP.NET applications.

Let's start with copying the files necessary for the CAPTCHA control (as
well as other custom controls used in the Subtext blog skin). As I noted in
my previous post, the CAPTCHA control requires an XML file for configuration,
a jQuery script (s3capcha.js) to replace the radio buttons with images (and
other runtime behavior), and the actual images themselves. Oh, and we obviously
need the Caelum assembly containing the code-behind for the user control as
well.

To copy these files into the corresponding folders in the **blog** application, I created a new MSBuild target named **CopySharedFilesToBlogApplication**
(by unloading the **Website** project in Visual Studio and then
editing the **Website.csproj** file):

```
<Target Name="CopySharedFilesToBlogApplication">
    <Message Importance="high"
      Text="TODO: Copy CAPTCHA control and other shared files to blog folder..." />
  </Target>
```

To ensure this target is executed as part of every build, I modified the
**BuildDependsOn** property by adding a new **PropertyGroup**:

```
<PropertyGroup>
    <BuildDependsOn>
      $(BuildDependsOn);
      CopySharedFilesToBlogApplication
    </BuildDependsOn>
  </PropertyGroup>
```

> **Note**
>
>       You might be tempted to use the **AfterBuild** target or 
>       a post-build event to implement additional build steps like the ones 
>       described in this post. However, in general, I don't recommend that 
>       (for reasons described in
>       [one of my previous blog posts](/blog/jjameson/2008/04/10/a-better-way-to-build-sharepoint-solution-packages-and-cab-files)).

If you were to reload the project file at this point and perform a build,
you would see the "TODO:" message in the **Output** window.

To specify the list of files needed by the CAPTCHA control, I defined a new
**ItemGroup**:

```
<ItemGroup>
    <CaptchaFile Include="Controls\Captcha\Captcha.ascx" />
    <CaptchaFile Include="Controls\Captcha\config.xml" />
    <CaptchaFile Include="Controls\Captcha\Scripts\s3Capcha.js" />
    <CaptchaFile Include="Controls\Captcha\**\*.jpg" />
  </ItemGroup>
```

Then I added a **Copy** task to copy the CAPTCHA files into
the corresponding locations under the **blog** folder (in other
words, by preserving the relative path of each file that is copied):

```
<Target Name="CopySharedFilesToBlogApplication">
    ...
    <Copy SourceFiles="@(CaptchaFile)"
      DestinationFolder="blog\%(CaptchaFile.RelativeDir)" />
  </Target>
```

To copy the assembly containing the code-behind for the CAPTCHA user control,
I added another **Copy** task that specifies `"$(OutDir)$(TargetFileName);"`
as the source file (which translates to **bin\TechnologyToolbox.Caelum.Website.dll**)
and `"blog\bin"` as the destination
folder:

```
<Target Name="CopySharedFilesToBlogApplication">
    ...
    <Copy SourceFiles="$(OutDir)$(TargetFileName);"
      DestinationFolder="blog\bin" />
  </Target>
```

To copy dependencies (e.g. the ELMAH assembly and other Caelum assemblies
referenced in the **Website** project), I added one more
**Copy** task:

```
<Target Name="CopySharedFilesToBlogApplication">
    ...
    <Copy SourceFiles="@(ReferenceCopyLocalPaths)"
      DestinationFiles="@(ReferenceCopyLocalPaths->'blog\bin\%(DestinationSubDirectory)%(Filename)%(Extension)')" />
  </Target>
```

Here is the completed target:

```
<Target Name="CopySharedFilesToBlogApplication">
    <Message Importance="high"
      Text="Copying CAPTCHA control and other shared files to blog folder..." />
    <Copy SourceFiles="@(CaptchaFile)"
      DestinationFolder="blog\%(CaptchaFile.RelativeDir)" />
    <Copy SourceFiles="$(OutDir)$(TargetFileName);"
      DestinationFolder="blog\bin" />
    <!-- copy referenced assemblies to blog\bin folder -->
    <Copy SourceFiles="@(ReferenceCopyLocalPaths)"
      DestinationFiles="@(ReferenceCopyLocalPaths->'blog\bin\%(DestinationSubDirectory)%(Filename)%(Extension)')" />
  </Target>
```

With these changes, whenever a file in the website project is updated, all
of the "shared" files are copied to the **blog** application when
a build is is performed in Visual Studio.

However, what about when the solution is built using Team Foundation Build,
and subsequently deployed from the **\_PublishedWebsites** folder?

To support that scenario, I created another task named **CopySharedFilesToBlogApplicationInWebProjectOutputDir**
(admittedly a mouthful, but so be it). This target is similar to the previous
one, except that it uses the `$(WebProjectOutputDir)`
variable to copy the files to the **\_PublishedWebsites** folder:

```
<!--
    The following target is used to copy the shared files to the
    _PublishedWebsites\{app}\blog folder (when building the solution using Team
    Build)
  -->
  <Target Name="CopySharedFilesToBlogApplicationInWebProjectOutputDir">
    <Message Importance="high"
      Text="Copying CAPTCHA control and other shared files to blog folder in Web project output directory..." />
    <Copy SourceFiles="@(CaptchaFile)"
      DestinationFolder="$(WebProjectOutputDir)\blog\%(CaptchaFile.RelativeDir)" />
    <Copy SourceFiles="$(OutDir)$(TargetFileName);"
      DestinationFolder="$(WebProjectOutputDir)\blog\bin" />
    <!-- copy referenced assemblies to _PublishedWebsites\{app}\blog\bin folder -->
    <Copy SourceFiles="@(ReferenceCopyLocalPaths)"
      DestinationFiles="@(ReferenceCopyLocalPaths->'$(WebProjectOutputDir)\blog\bin\%(DestinationSubDirectory)%(Filename)%(Extension)')" />
  </Target>
```

To integrate this target into the TFS build process, I hook into the
**\_CopyWebApplication** target:

```
<PropertyGroup>
    <BuildDependsOn>
      $(BuildDependsOn);
      CopySharedFilesToBlogApplication
    </BuildDependsOn>
    <OnAfter_CopyWebApplication>
      CopySharedFilesToBlogApplicationInWebProjectOutputDir
    </OnAfter_CopyWebApplication>
  </PropertyGroup>
```

