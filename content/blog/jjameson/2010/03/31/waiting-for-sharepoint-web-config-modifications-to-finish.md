---
title: Waiting for SharePoint Web.config Modifications to Finish
date: 2010-03-31T08:38:00-06:00
excerpt:
  This week I finally got around to fixing a bug that would occasionally occur
  when deploying our solution based on Microsoft Office SharePoint Server (MOSS)
  2007. 
   In the solution we use a variety of different features to configure different
  aspects...
aliases:
  [
    "/blog/jjameson/archive/2010/03/30/waiting-for-sharepoint-web-config-modifications-to-finish.aspx",
    "/blog/jjameson/archive/2010/03/31/waiting-for-sharepoint-web-config-modifications-to-finish.aspx",
  ]
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/03/31/waiting-for-sharepoint-web-config-modifications-to-finish.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/03/31/waiting-for-sharepoint-web-config-modifications-to-finish.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

This week I finally got around to fixing a bug that would occasionally occur
when deploying our solution based on Microsoft Office SharePoint Server (MOSS)
2007.

In the solution we use a variety of different features to configure different
aspects of the site (for example, creating a /Public site and configuring the
default page for that site with a login Web Part as well as some initial
content). We also use features to add Web.config elements, such as those
required for configuring Forms-Based Authentication and integrating with various
Web services.

One of the problems that I encountered during the first deployment to the
Production environment (PROD) is that when
[activating the features through a script](/blog/jjameson/2009/09/28/sample-walkthrough-of-the-dr-dada-approach-to-sharepoint),
the following error occurred sporadically:

{{< blockquote "font-italic text-danger" >}}

A web configuration modification operation is already running.

{{< /blockquote >}}

As a workaround, I simply activated the remaining features manually (initially
by directly invoking {{< kbd "stsadm.exe -o activatefeature" >}}, but later by
making a temporary copy of the script and removing the features that had already
been successfully activated). Note that the error only occurred in PROD (at
least in the beginning) and therefore we didn't consider it a high priority to
fix.

Months went by and each time the solution was deployed to PROD, the workaround
would be used whenever the error occurred.

I thought about putting a simple hack in the code to sleep for 5 or 10 seconds
(assuming the Web.config modifications would finish running in that amount of
time), but I never implemented that hack since my preference was to fix the
fundamental problem instead.

This week, I had some time to tackle the problem so I modified my local
development environment to more closely mimic PROD (i.e. by adding a second
front-end Web server to the farm). Consequently I was able to reproduce the bug
on a consistent basis.

When there are multiple front-end Web servers in the SharePoint farm, we need to
wait for the timer job that performs the Web.config modifications to complete
before continuing. Otherwise, the "configuration modification operation is
already running" error may occur when applying Web.config changes from two
different features in rapid succession.

To avoid the error, I added a little bit of code to the
**[SharePointWebConfigHelper](/blog/jjameson/2010/03/23/introducing-the-sharepointwebconfighelper-class)**
class in the **ApplyWebConfigModifications** method:

```
            if (webApp.Farm.TimerService.Instances.Count > 1)
            {
                // HACK:
                //
                // When there are multiple front-end Web servers in the
                // SharePoint farm, we need to wait for the timer job that
                // performs the Web.config modifications to complete before
                // continuing. Otherwise, we may encounter the following error
                // (e.g. when applying Web.config changes from two different
                // features in rapid succession):
                //
                // "A web configuration modification operation is already
                // running."
                //
                SharePointTimerJobHelper.WaitForOnetimeJobToFinish(
                   webApp.Farm,
                   "Windows SharePoint Services Web.Config Update",
                   20);
            }
```

As you can see, most of the work is delegated to the
**SharePointTimerJobHelper** class. **SharePointWebConfigHelper** only knows the
name of the one-time timer job (`"Windows SharePoint Services Web.Config Update"`) and a reasonable amount of time that the timer job should take to
complete (20 seconds). Note that it doesn't necessarily wait the full 20 seconds
for the timer job to complete (essentially the original hack that I thought
about implementing). Rather it waits at most 20 seconds for the timer job to
complete. Also note that there is no guarantee that the timer job actually
finished (or even ran) when the call to
`SharePointTimerJobHelper.WaitForOnetimeJobToFinish` returns. However, in my
testing I found that 20 seconds seemed like a good choice for the
<var>maximumWaitTime</var> parameter.

Here is the code for the **SharePointTimerJobHelper** class:

```
using System;
using System.Diagnostics;
using System.Globalization;
using System.Threading;

using Microsoft.SharePoint.Administration;

using Fabrikam.Demo.CoreServices.Logging;

namespace Fabrikam.Demo.CoreServices.SharePoint
{
    /// <summary>
    /// Exposes static methods for commonly used helper functions for SharePoint
    /// timer jobs. This class cannot be inherited.
    /// </summary>
    /// <remarks>
    /// All methods of the <b>SharePointTimerJobHelper</b> class are static and can
    /// therefore be called without creating an instance of the class.
    /// </remarks>
    [CLSCompliant(false)]
    public static class SharePointTimerJobHelper
    {
        private static bool IsJobDefined(
            SPFarm farm,
            string jobTitle)
        {
            Debug.Assert(farm != null);
            Debug.Assert(string.IsNullOrEmpty(jobTitle) == false);

            SPServiceCollection services = farm.Services;

            foreach (SPService service in services)
            {
                foreach (SPJobDefinition job in service.JobDefinitions)
                {
                    if (string.Compare(
                        job.Title,
                        jobTitle,
                        StringComparison.OrdinalIgnoreCase) == 0)
                    {
                        return true;
                    }
                }
            }

            return false;
        }

        /// <summary>
        /// Determines whether the specified timer job is currently running (or
        /// scheduled to run).
        /// </summary>
        /// <param name="farm">The farm to check if the job is running on.</param>
        /// <param name="jobTitle">The title of the timer job.</param>
        /// <returns><c>true</c> if the specified timer job is currently running
        /// (or scheduled to run); otherwise <c>false</c>.</returns>
        public static bool IsJobRunning(
            SPFarm farm,
            string jobTitle)
        {
            Debug.Assert(farm != null);
            Debug.Assert(string.IsNullOrEmpty(jobTitle) == false);

            SPServiceCollection services = farm.Services;

            foreach (SPService service in services)
            {
                foreach (SPRunningJob job in service.RunningJobs)
                {
                    if (string.Compare(
                        job.JobDefinitionTitle,
                        jobTitle,
                        StringComparison.OrdinalIgnoreCase) == 0)
                    {
                        return true;
                    }
                }
            }

            return false;
        }

        /// <summary>
        /// Waits for a one-time SharePoint timer job to finish.
        /// </summary>
        /// <param name="farm">The farm on which the timer job runs.</param>
        /// <param name="jobTitle">The title of the timer job (e.g. "Windows
        /// SharePoint Services Web.Config Update").</param>
        /// <param name="maximumWaitTime">The maximum time (in seconds) to wait
        /// for the timer job to finish.</param>
        public static void WaitForOnetimeJobToFinish(
            SPFarm farm,
            string jobTitle,
            byte maximumWaitTime)
        {
            if (farm == null)
            {
                throw new ArgumentNullException("farm");
            }

            if (jobTitle == null)
            {
                throw new ArgumentNullException("jobTitle");
            }
            else if (string.IsNullOrEmpty(jobTitle) == true)
            {
                throw new ArgumentException(
                    "The job title must be specified.",
                    "jobTitle");
            }

            float waitTime = 0;

            do
            {
                bool isJobDefined = IsJobDefined(
                    farm,
                    jobTitle);

                if (isJobDefined == false)
                {
                    Logger.LogDebug(
                        CultureInfo.InvariantCulture,
                        "The timer job ({0}) is not defined. It may have"
                            + " been removed because the job completed.",
                        jobTitle);

                    break;
                }

                bool isJobRunning = IsJobRunning(
                    farm,
                    jobTitle);

                Logger.LogDebug(
                    CultureInfo.InvariantCulture,
                    "The timer job ({0}) is currently {1}. Waiting"
                        + " for the job to finish...",
                    jobTitle,
                    isJobRunning == true ? "running" : "idle");

                const int sleepTime = 500; // milliseconds

                Thread.Sleep(sleepTime);
                waitTime += (sleepTime / 1000.0F); // seconds

            } while (waitTime < maximumWaitTime);

            if (waitTime >= maximumWaitTime)
            {
                Logger.LogWarning(
                    CultureInfo.InvariantCulture,
                    "Waited the maximum amount of time ({0} {1}) for the"
                        + " one-time job ({2}) to finish.",
                    maximumWaitTime,
                    maximumWaitTime == 1 ? "second" : "seconds",
                    jobTitle);
            }
            else
            {
                Logger.LogInfo(
                    CultureInfo.InvariantCulture,
                    "Waited {0} {1} for the one-time job ({2}) to finish.",
                    waitTime,
                    waitTime == 1 ? "second" : "seconds",
                    jobTitle);

            }
        }
    }
}
```

When applying Web.config changes through a feature, messages similar to the
following are logged:

{{< log-excerpt >}}

```
Verbose: Applying Web.config modifications to Web application (SharePoint - fabrikam-test80)...
Verbose: The timer job (Windows SharePoint Services Web.Config Update) is currently idle. Waiting for the job to finish...
...
Verbose: The timer job (Windows SharePoint Services Web.Config Update) is currently running. Waiting for the job to finish...
...
Verbose: The timer job (Windows SharePoint Services Web.Config Update) is currently idle. Waiting for the job to finish...
...
Verbose: The timer job (Windows SharePoint Services Web.Config Update) is not defined. It may have been removed because the job completed.
Information: Waited 15 seconds for the one-time job (Windows SharePoint Services Web.Config Update) to finish.
Information: Successfully applied Web.config modifications to Web application (SharePoint - fabrikam-test80).
```

{{< /log-excerpt >}}

In this particular instance, it took 15 seconds for the Web.config changes to be
applied and the corresponding one-time SharePoint timer job to be cleaned up.
However, this trace was captured from a VM in my home lab. I suspect PROD will
actually be significantly faster.
