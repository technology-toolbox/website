---
title: "Development and Build Environments"
date: 2009-09-25T01:57:00-07:00
excerpt: "In a previous post , I briefly touched on the \"DEV-TEST-PROD\" triad of environments that I typically recommend (at a minimum) for every organization doing any form of software development. 
 This post describes, in greater detail, the various environments..."
aliases: ["/blog/jjameson/archive/2009/09/25/development-and-build-environments.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/25/development-and-build-environments.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/25/development-and-build-environments.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

In a previous [post](/blog/jjameson/2009/06/09/environment-naming-conventions), I briefly touched on the "DEV-TEST-PROD" triad of environments that         I typically recommend (at a minimum) for every organization doing any form of software         development.

This post describes, in greater detail, the various environments used for developing,         building, testing, and deploying a solution. [If you've ever worked on a project         with me, you'll find this content is typically found in the Development Plan deliverable.]

When developing custom solutions, I typically recommend a minimum of four environments:

- Local Developer Environments (LOCAL)
- Development Integration Environment (DEV)
- Test Environment (TEST)
- Production Environment (PROD)

Since it is assumed that members of the Development team already have individual         workstations or laptops, I typically refer to the abbreviated "DEV-TEST-PROD" triad         when referencing the various environments.

> **Note**
>
> For very large development efforts, there may be multiple physical environments corresponding to DEV, TEST, and PROD. For example, numerous "labs" are used within Microsoft for testing various configurations of Windows. However this is not always necessary depending this size of a project. PROD might also refer to multiple physical locations (i.e. for disaster recovery) and depending on release schedules, an additional Maintenance Environment (MAINT) might also be necessary.

In addition to describing each environment, this post includes descriptions of the         additional servers that are necessary to support the development process. These         additional servers include the following:

- Build Server
- Source Control Server
- Release Server

### Local Developer Environments (LOCAL)

The "LOCAL" environment refers to the standard machine configuration (or virtual         machine configuration) for each developer. It is recommended that this environment         have all the applications required for the solution and all the tools required for         building and debugging the solution, as well as the standard tools that are needed         on the project (such as Microsoft Office).

> **Note**
>
> Developers must have administrative privileges on their own local environments in order to facilitate development and debugging.

Most of the functionality for the solution should be able to run in a LOCAL environment.         This allows the team members to work independently on their areas of the solution,         without having to compete for a single environment resource. The majority of a developer's         assigned functionality should be able to be coded and unit tested in his or her         own LOCAL environment. This reduces the contention in larger teams over access to         the environments.

There should be a minimum number of differences between the solution running in         a developer's LOCAL environment and the solution running in the DEV, TEST, or PROD         environments. Certain differences are expected, however, as there may be some changes         required in order to reduce costs or to emulate the behavior of a shared resource.

Some organizations use the term "sandbox" instead of LOCAL when referring to this         environment. I prefer the term LOCAL since it more accurately reflects the intent         of the environment and does not imply that the environment is completely isolated         (for example, when integrating with external systems, each developer's LOCAL environment         is often configured to connect to the systems corresponding to the DEV environment).

### Development Integration Environment (DEV)

The "DEV" environment is similar to each developer's LOCAL environment. The primary         purpose of DEV is to provide an integration environment where the latest build of         the complete solution can be evaluated, including all of the various pieces contributed         by various members of the Development team. DEV should have all the applications         used in the solution and the tools to debug them.

> **Note**
>
> This environment is owned by the Development team. All members of the Development team (or some subset, depending on the size and experience of the team) have administrative privileges in the DEV environment. Having administrative privileges allows developers to easily debug and reconfigure the environment.

After the Build Server finishes compiling the source code, the solution is installed         on the DEV environment. This is the first location where the installation process         is tested. It is also an important platform for ensuring the basic functionality         of the solution works.

The DEV environment is important for catching problems early on. Any project stakeholder,         not just members of the Development team, can use the DEV environment to evaluate         the progress of the solution.

> **Important**
>
> No development should occur on the DEV environment (in other words, no files should ever be checked out directly to the DEV environment). The DEV environment is shared by the entire team and must be treated appropriately as a shared resource. Careful consideration must be made when debugging in this environment (since this will likely interrupt anyone else accessing the DEV environment). Generally speaking, debugging should only take place on the DEV environment when the issue cannot be reproduced in a LOCAL environment. Notification should be provided to the entire team prior to initiating a debugging session directly on the DEV environment (this will also greatly reduce the time required to investigate the issue by avoiding concurrent activities).

### Build Server

The Build Server compiles all the code in the solution and creates the installation.         It should have the same configuration as a Local Developer Environment (LOCAL).         The Build Server should be dedicated to building the solution and should not contain         any unnecessary applications or tools on it. The machine should be kept as clean         and stable as possible so that it is in a good state to perform the builds.

All official builds that are distributed to be tested must be built on the Build         Server.

> **Note**
>
> Depending on the size of the organization and sophistication of the automated build process, a subset of the Development team may need administrative privileges on this server.

### Source Control

        Server

The source control system should be deployed on a server whose primary purpose is         to be the central repository for the source code. This should be dedicated hardware         that has sufficient resources to avoid bottlenecks when numerous team members are         working concurrently (e.g. 100Mb network connectivity between the source server         and the development machines).

The source control server should perform maintenance checks on a regular basis.         Backups should be scheduled at least once daily to ensure that the files can be         recovered in the event of a failure. Backups should be kept in a separate location         to ensure they can be recovered in case of hardware failure.

> **Note**
>
> The Development team has limited read-write privileges on this server (i.e. the minimum required for the source control system to function properly).

### Release Server

The Release Server keeps an archive of the output from each successful build. Under         the share that is used to archive the solution, there is a folder corresponding         to each build. Each build folder contains all of the files necessary to install         the solution. There is typically a folder for each project configuration in the         solution (e.g. Debug, Release, etc.).

For example:

> \\BuildServer\Builds\Fabrikam\Project1\1.0.1.0\Debug

In addition to the folders corresponding to each build, there is a folder named         **Latest** that always contains the most recent successful build. This         simplifies the process of installing the solution to the DEV environment.

> **Note**
>
> All team members should have read-only access to the Builds share. Only the service account used for the build process should have read-write access to the Builds share.

The shared folder on the Release Server should be protected by ACLs to ensure that         only members of the solution team can access the shares.

> **Important**
>
> The Test and Release Management teams should only install the solution from the Release Server (or, preferrably, a CD or DVD containing the files from the Release Server). Furthermore, only explicit build folders should be used by the Test and Release Management teams. In other words, the **Latest** folder should only be used for automated installs into the DEV environment -- it should not be used for installations in the TEST or PROD environments.

### Test Environment

        (TEST)

This environment should mirror the same configuration as the production environment         as much as possible. Having an exact replica of the deployed environment ensures         that the deployment is correct and repeatable when it goes live. In order to minimize         costs, TEST may not always have the same level of fault tolerance as the production         environment.

The primary purpose of TEST is to provide the Test team with an isolated environment         for validating test cases. In addition to functional testing, TEST is also used         for performance testing and "soak" testing (long-running test scenarios primarily         intended to identify resource leaks). Provided the differences between TEST and         PROD are only regarding true fault tolerance (and not scale), then performance testing         in TEST should accurately predict the performance of PROD.

> **Note**
>
> This environment is owned by the Test team. No member of the Development team should have privileges in TEST above those of the end users of the solution. Members of the Test team should have the additional privileges necessary to install the solution and assist the Development team with debugging issues in the TEST environment.

Typically, testers create their test cases and initially regress defects using the         DEV environment, however the formal validation of each test case must be performed         using the TEST environment.

Having a separate environment is valuable because it allows the Test team to decide         which specific build of the solution should be tested at any given time. "Smoke"         tests are typically performed on each build in DEV. Periodically the Test team selects         a successful build to promote to TEST.

### 

        Production Environment (PROD)

The Production environment is where the solution is ultimately deployed for end         users. Strict security procedures must be enforced in this environment.

> **Note**
>
> Only the Release Management team should have privileges in PROD above those of the end users of the solution.

