---
title: Introducing the SharePointWebConfigHelper Class
date: 2010-03-23T08:26:00-06:00
excerpt:
  Here is another helper class that I developed that you may find useful when
  building solutions for Windows SharePoint Services (WSS) v3 and Microsoft
  Office SharePoint Server (MOSS) 2007. If you use the SPWebConfigModification
  class to add or modify...
aliases:
  [
    "/blog/jjameson/archive/2010/03/22/introducing-the-sharepointwebconfighelper-class.aspx",
    "/blog/jjameson/archive/2010/03/23/introducing-the-sharepointwebconfighelper-class.aspx",
  ]
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/03/23/introducing-the-sharepointwebconfighelper-class.aspx"
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/03/23/introducing-the-sharepointwebconfighelper-class.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/03/23/introducing-the-sharepointwebconfighelper-class.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Here is another helper class that I developed that you may find useful when
building solutions for Windows SharePoint Services (WSS) v3 and Microsoft Office
SharePoint Server (MOSS) 2007.

If you use the
**[SPWebConfigModification](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.administration.spwebconfigmodification.aspx)**
class to add or modify Web.config files, you might have code that looks
something like this:

```
        private static void AddAuthenticationWebConfigModifications(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SPWebConfigModification ensureFormsNodeModification =
                new SPWebConfigModification(
                    "forms",
                    "configuration/system.web/authentication");

            ensureFormsNodeModification.Owner =
                fbaWebConfigModificationOwner;

            ensureFormsNodeModification.Sequence = 0;
            ensureFormsNodeModification.Type =
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode;

            ensureFormsNodeModification.Value =
                "<forms />";

            webApp.WebConfigModifications.Add(ensureFormsNodeModification);

            SPWebConfigModification defaultUrlModification =
                new SPWebConfigModification(
                    "defaultUrl",
                    "configuration/system.web/authentication/forms");

            defaultUrlModification.Owner =
                fbaWebConfigModificationOwner;

            defaultUrlModification.Sequence = 0;
            defaultUrlModification.Type =
SPWebConfigModification.SPWebConfigModificationType.EnsureAttribute;

            defaultUrlModification.Value = "/";

            webApp.WebConfigModifications.Add(defaultUrlModification);

            SPWebConfigModification timeoutModification =
                new SPWebConfigModification(
                    "timeout",
                    "configuration/system.web/authentication/forms");

            timeoutModification.Owner =
                fbaWebConfigModificationOwner;

            timeoutModification.Sequence = 0;
            timeoutModification.Type =
SPWebConfigModification.SPWebConfigModificationType.EnsureAttribute;

            timeoutModification.Value = "60"; // in minutes

            webApp.WebConfigModifications.Add(timeoutModification);
        }
```

There is quite a bit of repetitive code in the sample above and therefore it
looks like a good candidate for refactoring.

Personally, I find the following equivalent to be much more readable:

```
        private static void AddAuthenticationWebConfigModifications(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                fbaWebConfigModificationOwner,
                "forms",
                "configuration/system.web/authentication",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<forms />");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                fbaWebConfigModificationOwner,
                "defaultUrl",
                "configuration/system.web/authentication/forms",
SPWebConfigModification.SPWebConfigModificationType.EnsureAttribute,
                "/");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                fbaWebConfigModificationOwner,
                "timeout",
                "configuration/system.web/authentication/forms",
SPWebConfigModification.SPWebConfigModificationType.EnsureAttribute,
                "60" /* in minutes */ );
        }
```

Notice that the code for instantiating new instances of the
**SPWebConfigModification** class is now performed by the custom
**SharePointWebConfigHelper** class:

```
using System;
using System.Globalization;

using Microsoft.SharePoint.Administration;

using Fabrikam.Demo.CoreServices.Logging;

namespace Fabrikam.Demo.CoreServices.SharePoint
{
    /// <summary>
    /// Exposes static methods for commonly used helper functions for SharePoint
    /// Web.config files. This class cannot be inherited.
    /// </summary>
    /// <remarks>
    /// All methods of the <b>SharePointWebConfigHelper</b> class are static and can
    /// therefore be called without creating an instance of the class.
    /// </remarks>
    [CLSCompliant(false)]
    public static class SharePointWebConfigHelper
    {
        /// <summary>
        /// Adds a Web.config modification to the specified Web application.
        /// </summary>
        /// <remarks>The caller is responsible for updating the Web application
        /// and subsequently applying the Web.config modifications.</remarks>
        /// <param name="webApp">The Web application to add the Web.config
        /// modification to.</param>
        /// <param name="owner">The owner of the Web.config modification.
        /// </param>
        /// <param name="name">The name of the attribute or element.</param>
        /// <param name="path">The XPath expression that is used to locate the
        /// node that is being operated on.</param>
        /// <param name="type">The type of modification to make.</param>
        /// <param name="value">The value of the Web.config element or
        /// attribute.</param>
        public static void AddWebConfigModification(
            SPWebApplication webApp,
            string owner,
            string name,
            string path,
            SPWebConfigModification.SPWebConfigModificationType type,
            string value)
        {
            if (webApp == null)
            {
                throw new ArgumentNullException("webApp");
            }

            if (owner == null)
            {
                throw new ArgumentNullException("owner");
            }
            else if (string.IsNullOrEmpty(owner) == true)
            {
                throw new ArgumentException(
                    "The owner must be specified.",
                    "owner");
            }

            if (name == null)
            {
                throw new ArgumentNullException("name");
            }
            else if (string.IsNullOrEmpty(name) == true)
            {
                throw new ArgumentException(
                    "The name must be specified.",
                    "name");
            }

            if (path == null)
            {
                throw new ArgumentNullException("path");
            }
            else if (string.IsNullOrEmpty(path) == true)
            {
                throw new ArgumentException(
                    "The path must be specified.",
                    "path");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Adding SPWebConfigModication ({0} - {1}/{2})"
                    + " to Web application ({3})...",
                owner,
                path,
                name,
                webApp.DisplayName);

            SPWebConfigModification modification = new SPWebConfigModification(
                name,
                path);

            modification.Owner = owner;
            modification.Sequence = 0;
            modification.Type = type;

            if (string.IsNullOrEmpty(value) == false)
            {
                modification.Value = value;
            }

            webApp.WebConfigModifications.Add(modification);
        }

        /// <summary>
        /// Applies all Web.config modifications to the Web application.
        /// </summary>
        /// <param name="webApp">The Web application to apply the Web.config
        /// modifications to.</param>
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Design",
            "CA1011:ConsiderPassingBaseTypesAsParameters")]
        public static void ApplyWebConfigModifications(
            SPWebApplication webApp)
        {
            if (webApp == null)
            {
                throw new ArgumentNullException("webApp");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Applying Web.config modifications to Web application ({0})...",
                webApp.DisplayName);

            webApp.WebService.ApplyWebConfigModifications();

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

            Logger.LogInfo(
                CultureInfo.InvariantCulture,
                "Successfully applied Web.config modifications to Web"
                    + " application ({0}).",
                webApp.DisplayName);
        }

        /// <summary>
        /// Removes all Web.config modifications with the specified owner from
        /// the Web application.
        /// </summary>
        /// <remarks>If any Web.config modifications are removed, changes to the
        /// Web application are subsequently applied.
        /// </remarks>
        /// <param name="webApp">The Web application to remove the Web.config
        /// modification from.</param>
        /// <param name="owner">The owner of the Web.config modifications to
        /// remove.</param>
        public static void RemoveWebConfigModifications(
            SPWebApplication webApp,
            string owner)
        {
            if (webApp == null)
            {
                throw new ArgumentNullException("webApp");
            }

            if (owner == null)
            {
                throw new ArgumentNullException("owner");
            }
            else if (string.IsNullOrEmpty(owner) == true)
            {
                throw new ArgumentException(
                    "The owner must be specified.",
                    "owner");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Removing Web.config modifications for owner ({0})"
                    + " from Web application ({1})...",
                owner,
                webApp.DisplayName);

            int numberOfModificationsRemoved = 0;

            for (int i = webApp.WebConfigModifications.Count - 1; i >= 0; i--)
            {
                if (webApp.WebConfigModifications[i].Owner == owner)
                {
                    Logger.LogDebug(
                        CultureInfo.InvariantCulture,
                        "Removing SPWebConfigModication ({0}/{1})...",
                        webApp.WebConfigModifications[i].Path,
                        webApp.WebConfigModifications[i].Name);

                    webApp.WebConfigModifications.RemoveAt(i);
                    numberOfModificationsRemoved++;
                }
            }

            if (numberOfModificationsRemoved == 0)
            {
                Logger.LogDebug(
                   CultureInfo.InvariantCulture,
                   "No Web.config modifications found to remove for owner ({0})"
                       + " from Web application ({1}).",
                   owner,
                   webApp.DisplayName);
            }
            else
            {
                webApp.Update();

                ApplyWebConfigModifications(webApp);

                Logger.LogInfo(
                    CultureInfo.InvariantCulture,
                    "Successfully removed {0} Web.config modifications"
                        + " for owner ({1}) from Web application ({2}).",
                    numberOfModificationsRemoved,
                    owner,
                    webApp.DisplayName);

            }
        }
    }
}
```

The helper class also provides methods to apply and remove the Web.config
modifications. However, be aware that there is currently a bug in the
**SPWebConfigModification** infrastructure in which Web.config modifications are
only removed from the Web.config file for the default zone (not, for example,
the Internet zone).

Be sure to specify a unique "owner" of your Web.config modifications, such as
the namespace of the class that adds them.

For some good details about using **SPWebConfigModification**, refer to the
following:

{{< reference
title="Alirezaei, Reza (2008). SPWebConfigModification's Top 6 Issues. 2008-01-05."
linkHref="http://blogs.devhorizon.com/reza/?p=459" >}}

> **Update 2010-03-31**
>
> I enhanced the original version of **SharePointWebConfigHelper** to
> [wait for the Web.config modifications to finish](/blog/jjameson/2010/03/31/waiting-for-sharepoint-web-config-modifications-to-finish)
> in the **ApplyWebConfigModifications** method. If you don't want or need this
> fix, you can remove the reference to
> **[SharePointTimerJobHelper](/blog/jjameson/2010/03/31/waiting-for-sharepoint-web-config-modifications-to-finish)**.
