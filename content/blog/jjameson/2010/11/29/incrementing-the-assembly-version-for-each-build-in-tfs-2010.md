---
title: "Incrementing the Assembly Version for Each Build in TFS 2010"
date: 2010-11-29T16:18:00-07:00
excerpt: "Update (2010-12-03) 
 If you are using gated check-ins, be sure to also read my follow-up post: Bypassing a Gated Check-in in TFS 2010 http://blogs.msdn.com/b/jjameson/archive/2010/12/03/bypassing-a-gated-check-in-in-tfs-2010.aspx 
 
 Earlier this..."
aliases: ["/blog/jjameson/archive/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "TFS"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/11/29/incrementing-the-assembly-version-for-each-build-in-tfs-2010.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

> **Update (2010-12-03)**
>
> If you are using gated check-ins, be sure to also read my follow-up post:
>
> {{< reference title="Bypassing a Gated Check-in in TFS 2010"
> linkHref="/blog/jjameson/2010/12/03/bypassing-a-gated-check-in-in-tfs-2010"
> linkText="http://blogs.msdn.com/b/jjameson/archive/2010/12/03/bypassing-a-gated-check-in-in-tfs-2010.aspx"
>
> > }}

Earlier this year, I wrote a
[post](/blog/jjameson/2010/03/25/incrementing-the-assembly-version-for-each-build)
that explains the process I use for incrementing the assembly version with each
build in Team Foundation Server. However, the process was originally developed
for TFS 2005 and as you probably know by now, the build process in TFS 2010 has
changed significantly.

In TFS 2005 and 2008, the build process was defined entirely in MSBuild (i.e.
TFSBuild.proj), whereas in TFS 2010 the bulk of the build process is based on
Windows Workflow Foundation. TFS 2010 uses the workflow defined in
DefaultTemplate.xaml for new projects created in TFS 2010, but uses the
UpgradeTemplate.xaml workflow for TFS projects upgraded from a previous version.

The process I explained before generally works with TFS 2010 when using the
UpgradeTemplate.xaml workflow (since that workflow is essentially just a wrapper
around the legacy TFSBuild.proj file). In fact, I've been using that process
(with a few tweaks) for several TFS projects since upgrading the
["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) to TFS
2010 last May.

Nevertheless, I've been wanting to dive into the "standard" TFS 2010 build
process workflow (i.e. DefaultTemplate.xaml) and determine a better way (going
forward) to automatically increment the assembly version with each build.

Note that depending on your specific requirements, there may already be a
solution out there for incrementing the assembly version with each build.

For example, Jim Lamb wrote a
[post](http://blogs.msdn.com/b/jimlamb/archive/2010/02/12/how-to-create-a-custom-workflow-activity-for-tfs-build-2010.aspx)
back in February that describes how to create a custom workflow activity to
achieve build numbers like "2009.11.18.1" -- and if this versioning scheme works
for you, then great (you can stop reading this post and go read Jim's post
instead)! Personally, I'm not a fan of date-based versioning schemes --
primarily because an assembly version like 2009.11.18.1 tells me nothing more
than when the build was compiled. In other words, is this particular version a
major release, a minor release, or perhaps just a patch? Looking just at the
version number, I have no idea.

Also, as noted in Jim's post, John Robbins wrote a
[post](http://www.wintellect.com/CS/blogs/jrobbins/archive/2009/11/09/tfs-2010-build-number-and-assembly-file-versions-completely-in-sync-with-only-msbuild-4-0.aspx)
about synchronizing the TFS build number with the assembly version. John's
approach uses a versioning scheme that is similar -- if not identical -- to the
one used by the Visual Studio team, in which the
[Major Version](/blog/jjameson/2009/04/03/best-practices-for-net-assembly-versioning)
and
[Minor Version](/blog/jjameson/2009/04/03/best-practices-for-net-assembly-versioning)
are fixed (e.g. 10.0), the
[Build Number](/blog/jjameson/2009/04/03/best-practices-for-net-assembly-versioning)
is computed automatically from the current date, and the
[Revision](/blog/jjameson/2009/04/03/best-practices-for-net-assembly-versioning)
is incremented for each build on a particular day.

While I like John's approach to assembly versioning (especially since it doesn't
require anything to be installed on the build server), I still prefer to be able
to control all four portions of the assembly version. Maybe it's just because
I'm a "control freak" or perhaps it's simply because I've been doing it a
certain way for the last ten years and it's hard to teach this old dog new
tricks ;-)

As I've mentioned before, I increment the Build Number portion of the assembly
version with each build on the "Main" branch, whereas I increment the Revision
portion of the assembly version with each build on the "QFE" branch (for
patches).

If this seems a little fuzzy, consider the project that I've been working on for
a little over a year now. Our first build (off the Main branch) was 1.0.1.0. The
second build was 1.0.2.0, and the third build was 1.0.3.0. By the time we were
nearing the end of Sprint-3, we were up to build 1.0.51.0. However shortly
before the Sprint-3 release to Production (or shortly after -- it's been so long
that I honestly can't remember which) we needed to fix a couple of issues.
Consequently, we checked in those code changes on the QFE branch. Looking at
[the **Builds** list that I maintain on the SharePoint team site](/blog/jjameson/2010/11/29/create-a-custom-quot-builds-quot-list-on-your-tfs-project-portal-a-k-a-sharepoint-team-site),
it appears that 1.0.51.3 was the last build for Sprint-3.

While I can't tell which iteration or milestone build 1.0.51.3 corresponds to
(simply by looking at the assembly version), I can tell that it was a QFE/hotfix
for the 1.0.51.0 release. From this, I can infer that the number of code changes
between the 1.0.51.0 build and the 1.0.51.3 build are minimal. Contrast this
with assembly versions where the Build Number portion is based on a date and the
Revision is incremented based on the number of builds performed on that
particular day. I can certainly see advantages and disadvantages to each
approach.

Let's suppose that, like me, you want to keep using the assembly versioning
scheme that I've used in the past. Here is how I recommend you increment the
assembly version in TFS 2010. [Note: While I certainly hope you take the time to
understand the changes made to the default workflow in order to increment the
assembly version with each build, if you simply want to jump straight to the
solution, refer to the attachment to this post.]

First, it is important to have a high-level understanding of the default
workflow used in TFS 2010.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/TFS-2010-DefaultTemplate-Overview-232x431.png"
alt="DefaultTemplate.xaml - Overview" height="431" width="232"
title="Figure 1: DefaultTemplate.xaml - Overview" >}}

This "collapsed" view of the workflow illustrates the following high-level steps
of the build process:

1. Get the Build
2. Update Drop Location
3. Run On Agent
4. Check In Gated Changes for CheckInShelveset Builds

For the purposes of this post, the most interesting aspect of the build process
is the separation of the "Update Drop Location" activities from the "Run On
Agent" activities. Let's take a quick look at the details of "Update Drop
Location":

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/TFS-2010-Update-Drop-Location-254x464.png"
alt="DefaultTemplate.xaml - Update Drop Location" height="464" width="254"
title="Figure 2: DefaultTemplate.xaml - Update Drop Location" >}}

As you can see, the portion of the workflow that updates the build number does
not run on the build agent. In other words, it happens before the "Run On Agent"
scope and therefore runs on the build *controller* -- not the build *agent*
(although, in the case of the Jameson Datacenter -- and, I suspect, most TFS
environments -- there is a single build agent running on the build controller).

Consequently, we are going to have to make some significant changes to the
workflow if we want to increment the assembly version using a similar process to
the one previously used for TFS 2005/2008. To understand why, let's review the
high-level steps that I use for specifying the assembly version:

1. The assembly version (e.g. 1.0.0.0) is specified in the
   [SharedAssemblyInfo.cs file](/blog/jjameson/2009/04/03/shared-assembly-info-in-visual-studio-projects)
   located in the same folder as the Visual Studio solution. Individual Visual
   Studio projects reference this shared file using the concept of
   ["linked files" in Visual Studio](/blog/jjameson/2009/04/02/linked-files-in-visual-studio-solutions).
   Note that the assembly version is not incremented with each build.
2. The assembly file version (e.g. 1.0.51.0) is specified in the
   AssemblyVersionInfo.cs file, which is also located in the same folder as the
   Visual Studio solution. Depending on whether we are building off the Main
   branch or one of the QFE branches, either the Build Number or Revision is
   incremented with each build. For now, let's assume we are building off the
   Main branch, so we want to increment the Build Number portion of the version
   number.
3. The actual process of incrementing the version is performed using the Version
   task from the
   [MSBuild Community Tasks Project](http://msbuildtasks.tigris.org/). The
   Version task actually uses a simple text file (e.g. AssemblyVersionInfo.txt)
   to specify/increment the assembly version and subsequently generate the
   corresponding C# or VB.NET file (e.g. AssemblyVersionInfo.cs).

Note that you don't have to use the custom Version task from the MSBuild
Community Tasks Project to increment the version number. If you'd rather write a
custom workflow activity (&aacute; la Jim Lamb's post that I referred to
earlier), go right ahead. In my case, (a) I've already installed the custom
MSBuild Community Tasks on my build server, and (b) I know how to use the
Version task and I know that it does what I need it to do. Yes, this does
require that I run a little bit of "old school" MSBuild script inside my "shiny"
new Team Build workflow, but I certainly don't have any objections with that.
[The goal is to get all of the "goodness" of the new workflow-based build
process, while still leveraging the ability to increment/control each of the
four parts of the assembly version.]

With that review out of the way, let's turn our attention to modifying the
workflow to automatically increment the assembly version. For the purposes of
this walkthrough, I'll be using a brand new team TFS project that I chose to
name **foobar2010**. [Hopefully you can come up with a better name for your TFS
project ;-) ]

First, start by branching
**$/foobar2010/BuildProcessTemplates/DefaultTemplate.xaml** to
**$/foobar2010/BuildProcessTemplates/CustomTemplate.xaml**.
[Whenever possible, we want to isolate our customizations, and personally I prefer Jim's recommendation of branching the OOTB workflow file rather than simply creating a copy (as recommended on [MSDN](http://msdn.microsoft.com/en-us/library/dd647551.aspx))
-- even if I never intend to merge the changes into DefaultTemplate.xaml.]

The next step is to move a couple of the existing workflow activities out of the
**Update Drop Location** sequence into a new sequence named **Update Build
Number**, as shown below.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/TFS-2010-Increment-Version-Step-1-225x600.png"
alt="CustomTemplate.xaml - Step 1" height="600" width="225"
title="Figure 3: CustomTemplate.xaml - Step 1" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/TFS-2010-Increment-Version-Step-1-279x744.png)

Next, move **Update Build Number** and **Update Drop Location** inside the **Run
On Agent** scope, as shown below:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/TFS-2010-Increment-Version-Step-2-166x600.png"
alt="CustomTemplate.xaml - Step 2" height="600" width="166"
title="Figure 4: CustomTemplate.xaml - Step 2" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/TFS-2010-Increment-Version-Step-2-255x921.png)

Note that we need to first initialize the workspace (and hence be running on the
build agent) before updating the build number so we can access the various
assembly version files (e.g. AssemblyVersionInfo.txt and
AssemblyVersionInfo.cs). Also note that we want to ensure that the label applied
to the source code in TFS matches the build number. Consequently, the **Update
Build Number** sequence is placed *before* the activity that labels the source
code.

In the out-of-the-box workflow, the **LabelName** variable is scoped to **Run On
Agent** (which is okay provided the build number is set prior to entering the
**Run On Agent** scope). Since we are now updating the build number *inside*
**Run On Agent**, we need to change the scope of the **LabelName** variable to
ensure the label matches the updated build number.

To change the scope of the LabelName variable:

1. Within the **Run On Agent** scope, expand **If CreateLabel**.
2. Select **Create and Set Label for non-Shelveset Builds**.
3. Click the **Variables** tab, select the **LabelName** row, and in the
   **Scope** column, select **Create and Set Label for non-Shelveset Builds.**

In addition to the source code label, we want the drop location on the Release
Server to match the build number (e.g. \\dazzler\Builds\foobar2010\1.0.1.0).
Therefore the **Update Drop Location** sequence needs to come *after* the
**Update Build Number** sequence. [Whether the **Update Drop Location** sequence
comes before or after the activity that labels the source code really doesn't
matter. To me, it simply "feels better" to label the source code as early as
possible during the build process.]

By default, the drop location is set to:

```
BuildDetail.DropLocationRoot + "\" + BuildDetail.BuildDefinition.Name + "\" + BuildDetail.BuildNumber
```

Personally, I don't really care which build definition was used to create a
particular build. The truth is I can infer this from the build number. If the
build number is something like 1.0.51.0, then the build was created by the
"daily build" (i.e. a build definition named "Automated Build - Main"). If the
build number is something like 1.0.51.3, then the build was created by a "QFE
build" (e.g. a build definition named "QFE Build - v1.0").

More importantly, I want to make it as easy as possible for the Test and Release
Management folks to find a specific build when deploying the solution.
Consequently, remove the "extraneous" folder by updating the **Set Drop
Location** activity (inside the sequence within **If DropBuild And Build Reason
is Triggered**) so the **DropLocation** is set to:

```
BuildDetail.DropLocationRoot + "\" + BuildDetail.BuildNumber
```

Next, set the build number using the assembly version specified in the
AssemblyVersionInfo.txt file. To do this, add a new **InvokeProcess** activity
at the beginning of the **Update Build Number for Triggered Builds** activity in
the **Update Build Number** sequence, and set the properties as follows:

{{< table class="small" >}}

| <br>                    Property<br>                 | <br>                    Value<br>                 |
| --- | --- |
|  Arguments  |  "/C type """ + SourcesDirectory + "\\Source\\AssemblyVersionInfo.txt"""  |
|  DisplayName  |  InvokeProcess to read AssemblyVersion from file  |
|  FileName  |  "cmd.exe"  |

{{< /table >}}

All I'm doing here is using a little command-prompt "trickery" to read the
contents of a file (using the
[`type` command](http://en.wikipedia.org/wiki/Type_%28command%29)). The file
contains a single line of text that specifies the assembly version (e.g.
"1.0.1.0" -- without the quotes). As a result, the assembly version is
subsequently available using the **stdOutput** variable of the InvokeProcess
activity.

Move the existing **Update Build Number** activity inside the InvokeProcess
activity (below the **stdOutput** variable box) and change the
**BuilderNumberFormat** property to **stdOutput**.

While I certainly don't expect any errors to occur with the InvokeProcess
activity, it's still a good idea to ensure proper error handling in our build
process. Therefore, add a **Throw** activity (below the **errOutput** variable
box) and set the **Exception** property to:

```
New Exception(errOutput)
```

Thus if any error happens to occur while reading the assembly version from the
specified file, our build will immediately fail.

At this point, we've managed to set BuildDetail.BuildNumber (to whatever is
specified in the AssemblyVersionInfo.txt file -- e.g. 1.0.1.0). Consequently, if
we check-in the changes to CustomTemplate.xaml at this point, create a build
definition using this new process template, and subsequently run a build,
everything should work as expected.

However, what would happen if we started another build? Since we haven't yet
implemented the pieces to actually increment the assembly version, the second
build would fail (because, thankfully, TFS doesn't allow two builds to specify
the same build number).

Let's modify the workflow to increment the assembly version...

Just below the InvokeProcess activity added earlier (inside the **Update Build
Number for Triggered Builds** activity), add a new **MSBuild** activity, and set
the properties as follows:

{{< table class="small" >}}

| <br>                    Property<br>                 | <br>                    Value<br>                 |
| --- | --- |
|  DisplayName  |  Increment AssemblyVersion for next build  |
|  Project  |  SourcesDirectory + "\\Source\\IncrementAssemblyVersion.proj"  |

{{< /table >}}

Next, create the actual MSBuild file to increment the assembly version
(IncrementAssemblyVersion.proj):

```
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003" ToolsVersion="3.5"
  DefaultTargets="IncrementAssemblyVersion">

  <Import Project="$(MSBuildExtensionsPath)\MSBuildCommunityTasks\MSBuild.Community.Tasks.Targets"/>
  <Import Project="$(MSBuildExtensionsPath)\Microsoft\VisualStudio\TeamBuild\Microsoft.TeamFoundation.Build.targets" />

  <PropertyGroup>
    <TeamFoundationVersionControlTool>&quot;$(VS100COMNTOOLS)..\IDE\tf.exe&quot;</TeamFoundationVersionControlTool>
  </PropertyGroup>

  <Target Name="IncrementAssemblyVersion">
    <Message Importance="high"
      Text="Checking out version files from source control..." />

    <Exec
      WorkingDirectory="$(BuildProjectFolderPath)"
      Command="$(TeamFoundationVersionControlTool) checkout AssemblyVersionInfo.txt AssemblyVersionInfo.cs"/>

    <Message Importance="high"
      Text="Incrementing the assembly version..." />

    <Version
      VersionFile="$(BuildProjectFolderPath)\AssemblyVersionInfo.txt"
      BuildType="Increment"
      RevisionType="None">
      <Output TaskParameter="Major" PropertyName="Major" />
      <Output TaskParameter="Minor" PropertyName="Minor" />
      <Output TaskParameter="Build" PropertyName="Build" />
      <Output TaskParameter="Revision" PropertyName="Revision" />
    </Version>

    <CreateProperty
      Value="$(Major).$(Minor).$(Build).$(Revision)">
      <Output TaskParameter="Value" PropertyName="IncrementedAssemblyVersion" />
    </CreateProperty>

    <Message Importance="high"
      Text="Updating version file ($(BuildProjectFolderPath)\AssemblyVersionInfo.cs) with incremented assembly version ($(IncrementedAssemblyVersion))..." />

    <AssemblyInfo
      CodeLanguage="CS"
      OutputFile="$(BuildProjectFolderPath)\AssemblyVersionInfo.cs"
      AssemblyFileVersion="$(IncrementedAssemblyVersion)" />

    <Message Importance="high"
      Text="Checking in version files to source control..." />

    <Exec
      WorkingDirectory="$(BuildProjectFolderPath)"
      Command="$(TeamFoundationVersionControlTool) checkin /override:&quot;Check-in from automated build&quot; /comment:&quot;Increment assembly version ($(IncrementedAssemblyVersion)) $(NoCICheckinComment)&quot; AssemblyVersionInfo.txt AssemblyVersionInfo.cs"/>

  </Target>
</Project>
```

If you are familiar with the MSBuild customizations described in my earlier
post, then this should seem very straightforward. In fact, to create this file,
I simply copied one of my previous TFSBuild.proj files and started removing the
parts that are no longer needed (because they are now implemented in the
workflow). Hence the "3.5" version of this MSBuild file. If you want to update
it to use MSBuild 4.0, go right ahead. Personally, I didn't see the need -- nor
did I want to "tempt fate" since I know the MSBuild Community Tasks work as
expected with the older version of MSBuild (specifically the Version task).

Be aware that the MSBuild file shown above is for the Main branch. For QFE
branches, I modify two lines in the file in order increment the `RevisionType`
instead of the `BuildType` (to generate assembly numbers like 1.0.51.1,
1.0.51.2, etc.).

Note that the **DisplayName** specified earlier for the new MSBuild activity is
"Increment AssemblyVersion for *next* build" (as opposed to something like
"Increment AssemblyVersion for *this* build"). This is an important point to
understand and warrants further explanation.

When TFS 2010 starts a build, it uses a specific changeset to identify what
version of the source code to get and compile. If, for example, a scheduled
build starts at 5:00:00 AM on Tuesday, but one of the developers (say, Jeremy)
happens to be working very early that morning and checks in code at 5:00:03 AM
(3 seconds after the build started), then Jeremy's changes are *not* included in
the build. In other words, we don't want any changesets included in the build
after the changeset specified for the build (i.e. the **GetVersion** that is
specified as an argument when starting the build).

Consequently, to ensure the incremented assembly version applies to the next
build (and not the current build), we need to "rollback" the changeset created
by the IncrementAssemblyVersion.proj MSBuild file. Fortunately, this is very
easy to do -- simply by sync'ing the workspace again.

Copy the existing **Get Workspace** activity (the last activity in the
**Initialize Workspace** sequence) and paste it below the new MSBuild activity
added previously (**Increment AssemblyVersion for next build**). To clarify the
purpose of this second SyncWorkspace activity, I recommend changing the
**DisplayName** to **Sync workspace to revert AssemblyVersion**.

At this point, CustomTemplate.xaml should look like the following (in order to
conserve space, only the **Update Build Number** portion is shown):

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/TFS-2010-Increment-Version-Step-3-292x586.png"
alt="CustomTemplate.xaml (Update Build Number) - Step 3" height="586"
width="292"
title="Figure 5: CustomTemplate.xaml (Update Build Number) - Step 3" >}}

That's it -- we're done!

Check in the CustomTemplate.xaml file and queue a new build. Wait a few minutes
for the build to complete and verify the new build appears as expected in the
drop location.

Lastly, before finishing off this post, I want to discuss a couple of potential
areas of concern. First, I've heard several people state that checking in files
as part of the build process is not considered a "best practice." I've even seen
some people go so far as to store their assembly version file (the equivalent of
my AssemblyVersionInfo.txt file) on a network share somewhere simply to avoid
checking out and subsequently checking in the updated file. Quite frankly, this
scares me...a lot.

One could certainly argue that recovering this file is fairly easy and reliable
(for example, in the event it is mistakenly deleted from the network share, you
can easily "reverse engineer" the file from the most recent build number).
However, just the mere idea of storing a file that is an integral part of your
build anywhere outside of source control is enough to send me running down the
hall yelling at the top of my lungs. It just doesn't feel right.

At the same time, however, creating a changeset as part of each "official" build
certainly doesn't come without penalty. For example, let's suppose that your
development project is fairly small and consequently the vast majority of your
development is done on the Main branch. This means that your developers will
likely be rebuilding the entire solution at least once per day (assuming you
schedule a "daily build" that increments the assembly version). This assumes
that each developer does a "Get Latest" each morning (which I certainly hope
they do, simply out of habit), which subsequently downloads the incremented
AssemblyVersionInfo.cs file (which is referenced by each and every project in
the solution). Consequently all of the assemblies are compiled during the next
build on the developer's local environment.

Honestly, in the ten years I've been using this approach, this "daily rebuild"
has never presented a significant problem. It just becomes part of the expected
"warmup" period each morning. In other words, it's a great excuse to go get a
cup of coffee or discuss last night's episode of
[In Treatment](http://en.wikipedia.org/wiki/In_Treatment) with your colleagues
hanging out in the break room ;-)

However, if you are building a very large solution that takes, say, something
like 10 minutes (or more) to rebuild then this could definitely become an issue
-- especially when you get close to a milestone and you start kicking off builds
throughout the day. In that case, though, you probably don't have everyone
working off your Main branch (that's certainly not how the DevDiv folks build
the .NET Framework and Visual Studio). Instead, you likely have "Development"
branches that people work on, and personally, I don't see any value in
incrementing the assembly version automatically when build Dev branches.

The other concern that I've heard with regards to checking in an updated file as
part of the build process is that it "pollutes" your merge information. For
example, consider the example where the Revision is incremented as part of the
QFE build process. Consequently, when performing a "reverse integration" (i.e.
merging a hotfix from the QFE branch back through to the Main branch), you have
to ignore the changes to the assembly version files. Again, at least in my
experience, this doesn't introduce a significant level of pain. Your pain
threshold may be much lower than mine in this particular area. If that's the
case, then perhaps you should use a different assembly versioning scheme. [Don't
worry, you are not going to hurt my feelings ;-) ]

In case you are wondering how I configure build definitions, here are the
settings for the "daily build" as an example. If a setting is not listed in the
following table, it means the default is used.

{{< table class="small"
caption="Build Definition: \"Automated Build - Main\"" >}}

| <br>                    Section<br>                 | <br>                    Property<br>                 | <br>                    Value<br>                 |
| --- | --- | --- |
|  General  |  Build definition name  |  Automated Build - Main  |
|  Trigger  | Schedule - build every week on the following days<ul><li>Monday</li><li>Tuesday</li><li>Wednesday</li><li>Thursday</li><li>Friday</li><li>Saturday</li><li>Sunday</li></ul> |  (selected)  |
|   |  Queue the build on the build controller at:  |  4:45 AM  |
|  Workspace  | <br>                    Source Control Folder<br><br>                    Build Agent Folder<br>                 | <br>                    $/foobar2010/Main<br><br>                    $(SourceDir)<br>                 |
|  Build Defaults  |  Copy build output to the following drop folder (UNC path, such as \\server\share):  |  \\dazzler\Builds\foobar2010  |
|  Process  |  Build process template:  |  CustomTemplate.xaml  |
|   |  Build process parameters:  |   |
|   | Items to Build<ul><li>Solutions/Projects</li><li>Configurations</li></ul> | <br><ul><li>$/foobar2010/Main/Source/foobar.sln</li><li>Debug - Any CPU<br><br>                            Release - Any CPU</li></ul> |
|  Retention Policy  | Triggered and Manual<ul><li>Succeeded<ul><li>Retention Policy</li></ul></li></ul> | <br><br><br>                    Keep All<br>                 |

{{< /table >}}

