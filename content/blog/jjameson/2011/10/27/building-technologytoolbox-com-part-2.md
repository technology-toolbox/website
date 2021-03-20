---
title: Setting up a new Web development project (a.k.a. Building TechnologyToolbox.com, part 2)
date: 2011-10-27T05:42:45-06:00
excerpt:
  Once I settled on using Subtext as the blogging solution for the Technology
  Toolbox site, I turned my attention to working on the other areas of the
  site...
aliases:
  [
    "/blog/jjameson/archive/2011/10/26/building-technologytoolbox-com-part-2.aspx",
    "/blog/jjameson/archive/2011/10/27/building-technologytoolbox-com-part-2.aspx",
  ]
draft: true
categories: ["Development", "My System"]
tags: ["My System", "TFS", "Visual Studio", "Web Development"]
---

Once I settled on
[using Subtext](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-1)
as the blogging solution for the Technology Toolbox site, I turned my attention
to working on the other areas of the site (as well as developing the custom blog
skin to make the pages rendered by Subtext appear seamless with the other pages
on the site).

However, I first needed to come up with a project name.

### "Caelum"

The only thing worse than a software project without a name is a boat without a
name. There's really no excuse for either one.

During my career, I have been involved in countless software projects. For some
of them I came up with the name (e.g. the "Insight" application for Gambro
Healthcare and the "Sagacity" project at Time-Warner Telecom) while others
already had established project names before I got involved (e.g. the "Frontier"
project at Agilent Technologies).

While you certainly don't want to spend a lot of time coming up with a project
name, I think it is definitely worth investing a little bit of effort in finding
an inspirational and meaningful name.

You are probably familiar with Microsoft code names based on places or locations
(e.g. "Cairo", "Chicago", "Everett", "Whidbey", "Hatteras", and "Currituck") or
celestial bodies and related items (e.g. "Jupiter" and "Voyager").

One source of code names that I've used in the past are the names of
constellations.

Thus when it came time to name the project for creating the new
TechnologyToolbox.com site, I took a look at the
[list of constellations on Wikipedia](http://en.wikipedia.org/wiki/List_of_constellations).
It didn't take long before I came across
"[Caelum](http://en.wikipedia.org/wiki/Caelum_%28constellation%29)" which means
"the chisel" in Latin.

Given the "Toolbox" portion of my company name, this sounded perfect! "Caelum"
would be the tool with which a new site is "sculpted" from nothingness into
something glorious. [Imagine trumpets blaring at this point...it's funnier.]

With that matter out of the way, I proceeded with the rest of the project setup.

### Create the project in Team Foundation Server

Once I had a project name ("Caelum"), the next step was to create a
corresponding TFS project. I chose to use the **MSF for Agile Software
Development v5.0** process template. Note that I use my "admin" account
(**TECHTOOLBOX\jjameson-admin**) to create TFS projects and complete other
configuration changes.

Next I bulk-loaded my initial list of work items using Microsoft Excel. I've
listed these initial tasks in a
[previous post](/blog/jjameson/2010/12/02/my-initial-thoughts-on-microsoft-visual-studio-scrum-1-0-tfs-2010-process-template),
in case you are interested in what they are.

The first task is to configure the permissions for the TFS project and the
second task is to configure permissions on the team project portal. Thus I went
through and quickly granted access to my "regular" account
(**TECHTOOLBOX\jjameson**) via a domain group (**TECHTOOLBOX\All Developers**).

### Create initial source tree and Visual Studio solution

At this point, I switched from my admin account to my regular account, created a
Visual Studio solution and a couple of projects (i.e. **CoreServices** and
**CoreServices.DeveloperTests**), and subsequently checked these in to TFS, as
illustrated in the following screenshot.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Caelum-Initial-source-tree-600x420.png"
alt="Caelum - Initial source tree and Visual Studio solution" class="screenshot"
height="420" width="600"
title="Figure 1: Caelum - Initial source tree and Visual Studio solution" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Caelum-Initial-source-tree-789x552.png)

{{< div-block "note" >}}

> **Note**
>
> I recommend creating folders like **Documents** (or "docs" if you prefer that
> instead), **References** (or "lib"), **Source** (or "src"), and **Tools**
> under the **Main** folder.
>
> Shortly after checking in this changeset, I realized that I forgot to add a
> **Source** folder under **Main** and put the solution in there -- instead of
> in the **Main** folder. Consequently I moved the items to the correct location
> and checked in the corresponding changeset.

{{< /div-block >}}

### Add custom dictionary to Visual Studio solution

Next, I added a custom dictionary to the Visual Studio solution, as described in
one of my earlier blog posts:

{{< reference
title="CA1704 Code Analysis Warning and Using Custom Dictionaries in Visual Studio"
linkHref="/blog/jjameson/2009/04/02/ca1704-code-analysis-warning-and-using-custom-dictionaries-in-visual-studio"
linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2009/04/02/ca1704-code-analysis-warning-and-using-custom-dictionaries-in-visual-studio.aspx" >}}

### Generate strong name key and configure assembly signing

For as long as I've been developing on the .NET Framework, I've always
recommended using a single "snk" file for all projects in a Visual Studio
solution. On the last consulting project I worked on before I left Microsoft,
the development team used a different key file for each project. Sure, that may
be the way Visual Studio 2010 behaves "out-of-the-box" with SharePoint projects,
but that doesn't mean it's a best practice. In my opinion, have a unique key
file per project is a pain in the neck.

For the Caelum solution, I created a single key file, added it to the solution
(as well as to the individual projects in the solution), and updated the
settings for each project to strong name the assembly using the specified key.

For more details about how to do this, refer to the following post:

{{< reference title="Linked Files in Visual Studio Solutions"
linkHref="/blog/jjameson/2009/04/03/shared-assembly-info-in-visual-studio-projects"
linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2009/04/03/shared-assembly-info-in-visual-studio-projects.aspx" >}}

### Create custom rule set and enable code analysis

People often say "it's the little things in life..."

One of the subtle changes in Visual Studio 2010 is the way code analysis rules
are stored. This was somewhat of a nightmare in Visual Studio 2008 when you
tried to compare two versions of a project file to see what changed in the code
analysis settings (due to the fact that the enabled rules were specified on a
single line in the project file).

Fortunately, in Visual Studio 2010, the list of code analysis rules that are
enabled for a particular project is stored outside of the project file (in a
.ruleset file).

Even better, you can easily create your own custom rule set that specifies *all*
code analysis rules should be enabled and any violations should be treated as
errors:

```
<?xml version="1.0" encoding="utf-8"?>
<RuleSet Name="TechnologyToolbox.Caelum.ruleset"
  Description="Custom rule set for the TechnologyToolbox.Caelum solution."
  ToolsVersion="10.0">
  <IncludeAll Action="Error" />
</RuleSet>
```

This is how I prefer to start out. I then disable specific rules if I find them
to be inapplicable and I can't otherwise easily disable them in
GlobalSuppressions.cs.

### Add "SharedAssemblyInfo" and "AssemblyVersionInfo" files to Visual Studio solution

Next I created **SharedAssemblyInfo.cs** and **AssemblyVersionInfo.cs** files
and added these to the Visual Studio solution and projects. You can read more
about these files in the following post:

{{< reference title="Shared Assembly Info in Visual Studio Projects"
linkHref="/blog/jjameson/2009/04/02/linked-files-in-visual-studio-solutions"
linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2009/04/02/linked-files-in-visual-studio-solutions.aspx" >}}

### Create custom Team Foundation Build workflow to increment the assembly version with each build

As described in one of my Top 10 most popular blog posts, I strongly recommend
that you increment the assembly version automatically as part of every build
performed on the Build Server (i.e. through TFS Build):

{{< reference title="Best Practices for .NET Assembly Versioning"
linkHref="/blog/jjameson/2009/04/03/best-practices-for-net-assembly-versioning"
linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2009/04/03/best-practices-for-net-assembly-versioning.aspx" >}}

If you are not sure how to do this, and you are using TFS 2010, refer to the
following post for step-by-step details on how to accomplish this:

{{< reference
title="Incrementing the Assembly Version for Each Build in TFS 2010"
linkHref="/blog/jjameson/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010"
linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010.aspx" >}}

### Configure TFS check-in policies

With the basic setup of the project completed, I switched back to my admin
account and configured a number of TFS check-in policies on the project, as
described in the following post:

{{< reference title="Recommended Check-In Policies for Team Foundation Server"
linkHref="/blog/jjameson/2009/10/31/recommended-check-in-policies-for-team-foundation-server"
linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2009/10/31/recommended-check-in-policies-for-team-foundation-server.aspx" >}}

If you are wondering why I didn't configure these check-in policies earlier, the
answer is that I do it this way in order to avoid issues when enabling code
analysis on the Visual Studio projects. For example if, like me, you configure
code analysis to treat all violations as errors (instead of warnings) then your
solution won't build if you haven't yet generated a strong name key and
configured assembly signing. And if your solution doesn't build (and you've
added a "Builds" check-in policy), then you can't check-in a changeset (unless
you override the warning, but I rarely choose to do that).

### Create Web site project

With a solid foundation in place, the final step was to create a new ASP.NET Web
project and add it to the solution. Note that like the **CoreServices** and
**CoreServices.DeveloperTests** projects, the **Website** project includes the
following linked files:

- AssemblyVersionInfo.cs
- SharedAssemblyInfo.cs
- CustomDictionary.xml
- TechnologyToolbox.Caelum.snk

Similarly, it is configured to treat all warnings as errors (under the **Build**
section in the project properties) and it uses the same code analysis rule set
described earlier.

Consequently, I had to make a number of changes to the default code generated by
Visual Studio in order to get the project to compile successfully.

Enabling code analysis (and choosing to treat all violations as errors instead
of warnings) definitely comes with a little "pain" in the beginning. However, as
you may have heard me say in the past, this is trivial compared to the pain of
enabling code analysis later on (after you have a substantial code base already
written).
