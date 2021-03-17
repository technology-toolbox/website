---
title: "Localization and SharePoint Solutions, Part 2 (a.k.a. the CurrentUICultureSwitcher class)"
date: 2011-04-04T04:33:00-06:00
excerpt: "In part 1 of this series , I mentioned how I've been involved in several SharePoint projects for large, multinational corporations including Agilent Technologies and KPMG . I also mentioned how one of the sprints last year for my current project was dedicated..."
aliases: ["/blog/jjameson/archive/2011/04/03/localization-and-sharepoint-solutions-part-2-a-k-a-the-currentuicultureswitcher-class.aspx", "/blog/jjameson/archive/2011/04/04/localization-and-sharepoint-solutions-part-2-a-k-a-the-currentuicultureswitcher-class.aspx"]
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "MOSS 2007", "SharePoint 2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/04/04/localization-and-sharepoint-solutions-part-2-a-k-a-the-currentuicultureswitcher-class.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/04/04/localization-and-sharepoint-solutions-part-2-a-k-a-the-currentuicultureswitcher-class.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In
[part 1 of this series](/blog/jjameson/2010/10/25/localization-and-sharepoint-solutions-part-1),
I mentioned how I've been involved in several SharePoint projects for large,
multinational corporations including
[Agilent Technologies](http://www.chem.agilent.com/) and
[KPMG](http://www.kpmg.com). I also mentioned how one of the sprints last year
for my current project was dedicated to creating a Spanish version of a "Client
Portal" based on Microsoft Office SharePoint Server (MOSS) 2007.

Eventually, I intend to create a localized version of the
[Tugboat sample site running in SharePoint Server 2010](/blog/jjameson/2011/04/02/web-standards-design-with-sharepoint-part-4),
but I haven't found time to work on that just yet. Heck, it was only last week
that I finally finished "Sprint-2" of the Tugboat project, which consisted of
upgrading the Tugboat sample from MOSS 2007 to SharePoint 2010. From a planning
perspective, I don't expect to provide a localized version of Tugboat until
"Sprint-4" (since an interim sprint is necessary to first migrate the static
HTML content to various SharePoint lists). Patience, please, it's coming ;-)

In the meantime, I wanted to share my **CultureUICultureSwitcher** class that
I've found to be very useful when creating localized solutions in SharePoint.

Localizing a solution (regardless of whether the solution is a SharePoint site
or some other .NET application) is primarily a matter of extracting various
strings from the code into corresponding resource files. However, the trick is
ensuring the strings are subsequently read from the right resource file. In
typical .NET solutions, this is automatically handled based on the value of
[`Thread.CurrentThread.CurrentUICulture`](http://msdn.microsoft.com/en-us/library/system.threading.thread.currentuiculture.aspx).
In other words, if CurrentUICulture is set to "es-ES" then the Resource Manager
will attempt to read localized strings from the Spanish resource files.

Depending on how you choose to localize your SharePoint architecture (refer to
part 1 for more on what I mean by this) you may need to explicitly tell the
framework which locale you want to use when looking up localized text.

Even if you decide to go the "SharePoint language pack" route (in other words,
creating localized SharePoint sites in which the administration pages render in
various languages), you may still need to explicitly set
`Thread.CurrentThread.CurrentUICulture` to specify the desired language.

For example, suppose that, like me, you use custom SharePoint features to
configure default content on a site.

If you always activate the features through **Site Settings** (i.e. by browsing
to the admin pages of the site), then everything works as expected (because
`Thread.CurrentThread.CurrentUICulture` is set to right locale based on the
context of the current SharePoint site).

However, what if, like me, you prefer to activate features via the command line
instead (e.g. using PowerShell of StsAdm.exe)? In that case,
`Thread.CurrentThread.CurrentUICulture` is always going to be set to your
operating system language/region ("en-US" in my case). That obviously isn't
going to retrieve the Spanish text when configuring a SharePoint site created in
the Spanish language.

Consequently we need a way to temporarily change the CurrentUICulture -- but
ensure that it gets properly reverted back to the original value regardless of
whether everything works as expected or some error occurs.

Enter the **CultureUICultureSwitcher** class...

```
using System;
using System.Diagnostics;
using System.Globalization;
using System.Threading;

namespace Fabrikam.Demo.CoreServices
{
    /// <summary>
    /// Temporarily changes the current culture used by the Resource Manager to
    /// look up culture-specific resources at run time.
    /// </summary>
    public class CurrentUICultureSwitcher : IDisposable
    {
        private CultureInfo previousUICulture;

        private bool disposed;

        /// <summary>
        /// Initializes a new instance of the
        /// <see cref="CurrentUICultureSwitcher"/> class and sets the current
        /// culture used by the Resource Manager to look up culture-specific
        /// resources at run time.
        /// </summary>
        /// <param name="culture">The culture to be used when looking up
        /// culture-specific resources.</param>
        public CurrentUICultureSwitcher(
            CultureInfo culture)
        {
            if (Thread.CurrentThread.CurrentUICulture != culture)
            {
                previousUICulture = Thread.CurrentThread.CurrentUICulture;

                Thread.CurrentThread.CurrentUICulture = culture;
            }
        }

        /// <summary>
        /// Attempts to free resources and perform other cleanup operations
        /// before the object is reclaimed by garbage collection.
        /// </summary>
        /// <remarks>
        /// Assuming all instances of the <see cref="CurrentUICultureSwitcher"/>
        /// class are properly disposed, the finalizer should never be invoked.
        /// </remarks>
        ~CurrentUICultureSwitcher()
        {
            Debug.Fail("CurrentUICultureSwitcher was not properly disposed.");
            Dispose(false);
        }

        #region IDisposable implementation

        /// <summary>
        /// Releases all resources associated with an instance of the class.
        /// </summary>
        public void Dispose()
        {
            Dispose(true);
            GC.SuppressFinalize(this);
        }

        /// <summary>
        /// Releases resources associated with an instance of the class.
        /// </summary>
        /// <param name="disposing"><c>true</c> if managed resources should be
        /// disposed; <c>false</c> if only unmanaged resources should be
        /// disposed.</param>
        protected virtual void Dispose(
            bool disposing)
        {
            if (disposed == false)
            {
                if (previousUICulture != null)
                {
                    Thread.CurrentThread.CurrentUICulture =
                        previousUICulture;
                }

                if (disposing)
                {
                    // Release all managed resources here
                }

                // Release all unmanaged resources here
            }

            disposed = true;
        }

        #endregion
    }
}
```

I wish I could have thought of a more creative name for this, but oh well. It
seems to o convey the point of the class.

Here's an example unit test that demonstrates how the class is expected to work:

```
        /// <summary>
        /// Basic test for CurrentUICultureSwitcher.
        /// </summary>
        [TestMethod()]
        public void CurrentUICultureSwitcherTest001()
        {
            const int defaultCultureLcid = 1033;
            const int newCultureLcid = 3082;

            Assert.AreEqual(
                defaultCultureLcid,
                CultureInfo.CurrentUICulture.LCID);

            CultureInfo newCulture = new CultureInfo(newCultureLcid);

            using (new CurrentUICultureSwitcher(newCulture))
            {
                Assert.AreEqual(
                    newCultureLcid,
                    CultureInfo.CurrentUICulture.LCID);
            }

            Assert.AreEqual(
                defaultCultureLcid,
                CultureInfo.CurrentUICulture.LCID);
        }
```

Lastly, here's an excerpt from a custom "Announcements" feature that shows how
the class is used to configure a localized SharePoint site:

```
        /// <summary>
        /// Creates and configures the "Announcements" site under the specified
        /// Web.
        /// </summary>
        /// <param name="parentWeb">An
        /// <see cref="Microsoft.SharePoint.SPWeb"/> object representing the
        /// parent Web of the "Announcements" site. This can either be the
        /// root Web ("/") or a language Web (e.g. "/es-ES").</param>
        public static void Configure(
            SPWeb parentWeb)
        {
            if (parentWeb == null)
            {
                throw new ArgumentNullException("parentWeb");
            }

            Logger.LogDebug(
                CultureInfo.CurrentCulture,
                "Configuring Announcements site under parent Web ({0})...",
                parentWeb.Url);

            // Change CurrentUICulture to ensure the "Announcements" site is
            // localized according to the language of the parent site.
            using (new CurrentUICultureSwitcher(parentWeb.Locale))
            {
                using (SPWeb announcementsWeb = EnsureAnnouncementsWeb(
                    parentWeb))
                {
                    ConfigureAnnouncementsWeb(announcementsWeb);
                }
            }

            Logger.LogInfo(
                CultureInfo.CurrentCulture,
                "Successfully configured Announcements site under parent"
                    + " Web ({0}).",
                parentWeb.Url);
        }
```

I hope you find the **CurrentUICultureSwitcher** class to be useful when
creating SharePoint solutions that need to support more than one language.
