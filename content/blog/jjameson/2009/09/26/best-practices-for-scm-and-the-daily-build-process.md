---
title: Best Practices for SCM and the Daily Build Process
date: 2009-09-26T11:12:00-06:00
description:
  In a previous post, I briefly discussed a simple branching strategy for Team
  Foundation Server (TFS). This was somewhat of a follow-up to another post in
  which I briefly referenced a great article titled The Importance of Branching
  Models in SCM . If...
aliases:
  [
    "/blog/jjameson/archive/2009/09/25/best-practices-for-scm-and-the-daily-build-process.aspx",
    "/blog/jjameson/archive/2009/09/26/best-practices-for-scm-and-the-daily-build-process.aspx",
  ]
categories: ["Development"]
tags: ["Core Development", "TFS"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/09/26/best-practices-for-scm-and-the-daily-build-process.aspx"
---

In a previous post, I briefly discussed
[a simple branching strategy for Team Foundation Server](/blog/jjameson/2009/02/10/branching-strategy-in-team-foundation-server)
(TFS). This was somewhat of a follow-up to another
[post](/blog/jjameson/2007/04/18/structure-visual-studio-solutions) in which I
briefly referenced a great article titled
[The Importance of Branching Models in SCM](http://downloads.seapine.com/pub/papers/SCMBranchingModels.pdf).
If you haven't read this article, I highly recommend it.

However, while branching is certainly an important aspect of Software
Configuration Management (SCM) -- and perhaps one of the most important aspects
-- it's certainly not the only one that warrants discussion. In this post, I
want to cover some other best practices for SCM as well as the daily build
process.

For example, if you are using TFS, you know that each TFS project typically has
a source control repository as well as a project team site in SharePoint (a.k.a.
the "Project Portal"). You also probably know that all SharePoint document
libraries can be configured to support versioning similar to TFS source control,
and thus allow you to, for example, view an old version of an installation guide
stored on the Project Portal. However, just because you _can_ do this doesn't
mean that it is _recommended_ as a best practice.

Note that I'm not saying you should not store any documents on the Project
Portal, nor am I saying that you should never enable versioning on document
libraries in the Project Portal.

On the contrary, the Project Portal provides a great place to store documents
that various project stakeholders need to access from time to time (without
going through Visual Studio or TFS Web Access), and I almost always recommend
enabling versioning on all document libraries in your SharePoint sites (provided
you have plenty of storage on the backend SQL Server, of course).

Enabling versions on document libraries provides the ability to go back and see
who changed a document and when (which is sort of like an "audit trail" on a
document). While you could use the SharePoint versioning feature to go "back in
time" in order to view the state of a document (e.g. an installation guide) for
a particular version or build of your solution, this would obviously involve a
good deal of considerable effort based on timestamps.

A much easier way of correlating a version of a particular document to a
particular build of your solution is to actually store the document in source
control within TFS. This allows you to quickly retrieve the document version
using a changeset number or, more generally, a build label.

### Build Labels

Back in the days of using Visual SourceSafe (VSS) on customer projects, at the
beginning of the build process, a label would always be applied to the project
in VSS. Since the build label was applied at the top-level project within a
branch, it applied to all files in the solution -- including the source code,
setup files, and automated tests. This ensures that the installation and tests
could be repeated for any particular build.

{{< div-block "note" >}}

> **Note**
>
> While VSS certainly provided the ability to retrieve a project based on a
> particular timestamp, it definitely wasn't easy (the only way that I was ever
> to do it was through
> [the SS command line utility](http://msdn.microsoft.com/en-us/library/asxkfzy4%28VS.80%29.aspx)).
> Thus build labels provided a quick way of getting a snapshot of the code for a
> particular build.
>
> With the concept of changesets in TFS -- and the ability to quickly get the
> code for a specific changeset -- build labels are obviously not as important
> as they were in VSS. However, regardless of which particular source control
> system you are using, build labels provide an easy way to identify and
> retrieve important builds (such as **Beta 1**, **Beta 2**, **RC1**, and
> **v1.0**) or simply specific versions of the solution (e.g. 1.0.57.0).

{{< /div-block >}}

### Source Code

Applying the build label to the entire solution allows the Development team to
obtain all of the source code for a particular build, thereby enabling them to
step through the code to debug an issue that may not be reproducible in the
latest version of the code.

### Setup Files

In addition to the source code, the files supporting the installation of the
solution (such as the install scripts and the installation guide -- but not the
compiled setup packages) should also stored in the source control system. In
this way, the changes to the installation can be tracked from one version to
another.

### Automated Tests

Automated tests change as features are added to the solution. Consequently the
tests must be matched to a specific build and therefore need to be checked into
the source control system (and therefore labeled as part of the correponding
build). Automated tests typically include unit tests as well as Build
Verification Tests (BVTs).

### Daily Build Process

The build process is a fundamental activity within any software project. In
order to ensure success, the build and deployment process needs to conform to a
number of basic requirements. If the solution cannot be built directly from its
source code then it is not possible to integrate many of the key processes
required for successful delivery.

The following sections describe the steps in the build process and who is
responsible for each part of the process:

- [Check-Ins](#Check-Ins)
- [Automated Build Process](#Automated_Build_Process)
- [Build Verification Tests](#Build_Verification_Tests)
- [Smoke Tests](#Smoke_Tests)
- [Investigating Failures](#Investigating_Failures)
- [Build Hand-off](#Build_Hand-off)

### Check-Ins {#Check-Ins}

All code included in a build must be checked into source control before the
build process is initiated. All checked-in code must compile and it is the
responsibility of the developer who checks in the code to ensure that the
solution builds and all files that are needed to build the solution are checked
in.

{{< div-block "note important" >}}

> **Important**
>
> The source control must not be left in a broken state at any time. If a build
> breaks, resolving the problem becomes the highest priority.

{{< /div-block >}}

{{< div-block "note" >}}

> **Tip**
>
> You can use the **Builds** check-in policy for TFS to ensure the solution
> compiles before a developer is allowed to check-in a changeset.

{{< /div-block >}}
Each member of the Development team is responsible for ensuring the following:
- Check-ins must not break functionality in the solution. If a developer checks
  in a change in their area of the code, he or she must not expect another
  developer to resolve errors caused by the check-in; either the person checking
  in the code must fix the dependant code when they check-in or the developer
  must work with the other developer(s) to coordinate the check-in.
- The installation process must also be kept in a functional state at all times.
  Whenever a developer adds a feature, he or she is responsible for making sure
  that any setup changes are completed as well. Feature owners are ultimately
  responsible for the installation and configuration of their features.
- The developer is responsible for writing any custom installation actions
  needed to deploy their feature, for knowing what files need to be deployed on
  which server, and what permissions the feature needs to run.
- The installation of the feature should be completed when the code is checked
  into source control. The development of any custom installation actions needs
  to be done in conjunction with the team lead(s), and should not be done in
  isolation.
- Changes to the installation should be checked in to give the team sufficient
  time to test the changes before the next scheduled build.
- Unit tests must pass in the local developer's environment (LOCAL) before a
  check-in is performed. If the unit tests are broken because of changes to the
  feature's functionality, the developer must resolve the failures -- or reach
  agreement with the rest of the Development team to disable certain tests until
  they can be fixed (which should be a rare occurrence).

{{< div-block "note" >}}

> **Tip**
>
> You can use the **Testing Policy** check-in policy for TFS to ensure that
> specific unit tests pass before a developer is allowed to check-in a
> changeset.

{{< /div-block >}}

If any of the above conditions cannot be met, the code should not be checked
into source control.

### Automated Build Process {#Automated_Build_Process}

The automated build process should:

1. Automatically increment the Build Number or Revision portion of the
   [assembly version](/blog/jjameson/2009/04/03/best-practices-for-net-assembly-versioning)
   (depending on which branch is being built)
1. Apply a build label (if appropriate)
1. Retrieve the code from the source control system
1. Compile the solution
1. Run any associated unit tests
1. Copy the build to the
   [Release Server](/blog/jjameson/2009/09/25/development-and-build-environments)

If an error occurs during the build -- including execution of the unit tests --
the entire process stops. In this event, an email detailing the error should be
mailed to the appropriate distribution list. Once the error is fixed, the build
can be restarted. Note that the build/revision number should be incremented for
the restarted build.

### Build Verification Tests {#Build_Verification_Tests}

Build Verification Tests (BVTs) are a quick, automated test run on the solution
to catch defects in the code. The BVTs should cover the major functionality of
the solution but are not meant to be a full regression of all the functionality
in the solution.

The time required to run the entire set of BVTs should be on the order of
minutes -- not hours. Otherwise the tests become too cumbersome to run on a
regular basis. [Note that this guideline obviously depends on the size of your
solution and the resources involved. For example, I believe the BVTs used by the
Windows team actually run for several hours, due to the size of the code base.
However, it is still short enough to repeat on a daily basis.]

BVTs are owned by the Test team in conjunction with the Development team to
verify the build has completed successfully and to catch any regressions that
are introduced as a result of changes to the solution. BVTs should be launched
immediately after the automated build and deployment process. The BVTs must
return an error to indicate if they have succeeded or failed. Only after the
BVTs have passed should the build be considered ready for additional testing by
the Test team.

As noted before, BVTs are owned by Test. The Test team should work closely with
Development to keep the BVTs in sync with the changes in the solution. By having
the BVTs owned by Test, it ensures that there is a knowledge transfer between
Development and Test for any changes to the behavior or architecture of the
solution.

### Smoke Tests {#Smoke_Tests}

In order to successfully deploy the solution, automated tests should be written
to survey the environments before and after the application has been deployed.
The smoke tests should check that the correct components have been installed,
the configuration of the solution has been completed correctly, and that the
network connectivity between machines is functioning correctly. The smoke tests
must result in a clear pass or fail.

Smoke tests are generally considered to be more exhaustive than BVTs. The key
differentiator is that BVTs are intended to identify whether a build is worthy
of additional testing. Ideally, a substantial portion of the testing performed
by the Test team is automated -- not manual. So whether you refer to your
automated tests as BVTs and consider them part of a "smoke test" is really just
a matter of semantics.

### Investigating Failures {#Investigating_Failures}

When a failure is found with the BVTs or automated smoke tests, the Test team
should be the first point of contact to ensure that the tests are correct and
the failure is valid.

### Build Hand-off {#Build_Hand-off}

The Release Management team should always lead the deployment of the solution,
with Test and Development provided support as necessary.

Note that the Development team owns the DEV environment. Consequently,
developers will often resort to "tweaking" the environment in order to get the
solution deployed and operational. However, in order for the solution to be
deployed to TEST, the configuration changes made in DEV must either automated as
part of the installation or thoroughly documented as part of the installation
guide.

### Troubleshooting Guide

When problems are discovered with the deployment of the solution, a
troubleshooting guide should be created (or updated) to capture the experience
learned in resolving problems in the solution. The troubleshooting guide is not
meant to be exhaustive in terms of troubleshooting all aspects of the solution;
however it should provide a baseline for developing the operational support
documents for maintaining the solution in the Production environment.

The troubleshooting guide should capture the process and tools used for
investigating the problems with the solution and where to look for logs, events,
etc. in the system. It should also capture patterns that have been seen in the
solution that point toward a particular fault in the system.

{{< div-block "note" >}}

> **Tip**
>
> A SharePoint site -- or even just a simple SharePoint list -- provides an
> excellent alternative to a Troubleshooting Guide document. Think of this site
> -- or list -- as a simple "[Knowledge Base](http://support.microsoft.com/)"
> for your solution.

{{< /div-block >}}
