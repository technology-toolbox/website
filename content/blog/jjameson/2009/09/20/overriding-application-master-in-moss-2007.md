---
title: "Overriding Application.master in MOSS 2007"
date: 2009-09-20T09:35:00-06:00
excerpt: "Microsoft Office SharePoint Server (MOSS) 2007 includes a variety of out-of-the-box master pages. Many are provided primarily as samples (e.g. BlueBand.master) and serve as a starting point for creating your own master page. There's also default.master..."
aliases: ["/blog/jjameson/archive/2009/09/19/overriding-application-master-in-moss-2007.aspx", "/blog/jjameson/archive/2009/09/20/overriding-application-master-in-moss-2007.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/09/20/overriding-application-master-in-moss-2007.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/09/20/overriding-application-master-in-moss-2007.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Microsoft Office SharePoint Server (MOSS) 2007 includes a variety of out-of-the-box master pages. Many are provided primarily as samples (e.g. BlueBand.master) and serve as a starting point for creating your own master page. There's also default.master which most people now instantly associate with SharePoint as soon as they see it. Note that when you create a new site based on the **Team Site** site template or the **Collaboration Portal** site template in MOSS 2007, your site is configured to use default.master for both site pages (e.g. default.aspx) as well as system pages (e.g. /{list}/Forms/AllItems.aspx).

You can easily change the master page used for both site pages as well as system pages by clicking the **Master page** link under the **Look and Feel** section on the **Site Settings** page.

However, note that your site also uses another master page -- application.master -- for application pages (e.g. /\_layouts/viewlsts.aspx). In MOSS 2007, there is no out-of-the-box way to configure application pages to use a different master page. This typically isn't a significant issue because application pages are usually only used by administrators of the site. However, there's at least one notable exception.

When you create a site (say, for example, /Library) using the **Document Center** site template, then you will notice that the default page (e.g. /Library/default.aspx) has a link to **View All Site Content** which refers to the "view lists" page (e.g. /Library/\_layouts/viewlsts.aspx). Consequently when any of your users click this link, they actually see a page that is rendered with application.master.

This is exactly the scenario I encountered on the Agilent Technologies project a couple of years ago.

When I researched this problem, I discovered that it was identified as a bug back in the Beta 2 days, but it was classified as "won't fix" since -- at least at that time -- it was only expected that site administrators would ever view pages rendered with application.master. In other words, the bug that I found didn't explicitly mention the **Document Center** site template.

Since we needed a solution for Agilent right away, I had to come up with a workaround, er, I mean *creative solution*. The following approach is based on the original solution that I developed for Agilent (but improved to use the master page configured through **Site Settings**, rather than specifying the master page in a configuration file).

In order to force pages that are hard-coded to use application.master (such as \_layouts/viewlsts.aspx) to use the same custom master page configured through **Site Settings**, a custom HttpHandler is used to explicitly set the master page on each request for an application page.

Here is the code for the custom HttpHandler:

```
namespace Fabrikam.Project1.PublishingLayouts.Web.UI
{
    public class ApplicationPageHandlerFactory : PageHandlerFactory
    {
        [EnvironmentPermission(SecurityAction.LinkDemand, Unrestricted = true)]
        public override IHttpHandler GetHandler(
            HttpContext context,
            string requestType,
            string url,
            string pathTranslated)
        {
            IHttpHandler pageHandler = base.GetHandler(
                context,
                requestType,
                url,
                pathTranslated);

            Debug.Assert(pageHandler != null, "pageHandler is null");

            Page page = pageHandler as Page;
            Debug.Assert(page != null, "Handler is not of type Page.");

            page.PreInit += new EventHandler(Page_PreInit);

            return pageHandler;
        }

        private void Page_PreInit(
            object sender,
            EventArgs e)
        {
            if (SPContext.Current == null)
            {
                Logger.LogWarning(
                    "ApplicationPageHandlerFactory::Page_PreInit - "
                    + "SPContext.Current is null"
                    + " (this is normal when deleting a site).");

                return;
            }

            SPWeb web = SPContext.Current.Web;

            if (web == null)
            {
                Logger.LogWarning(
                    "ApplicationPageHandlerFactory::Page_PreInit - "
                    + "SPContext.Current.Web is null.");

                return;
            }

            Page page = (Page)sender;
            if (page == null)
            {
                Debug.Fail("sender is null.");

                // If this actually happens in a Release build, simply log a
                // warning, but don't throw an exception
                Logger.LogWarning(
                    "ApplicationPageHandlerFactory::Page_PreInit - "
                    + "sender is null.");

                return;
            }

            bool overrideMasterPage = ShouldOverrideMasterPage(page);

            if (overrideMasterPage == false)
            {
                return;
            }

            OverrideMasterPageFromSharePointSite(web, page);
        }

        private static void OverrideMasterPageFromSharePointSite(
            SPWeb web,
            Page page)
        {
            Debug.Assert(web != null);
            Debug.Assert(page != null);

            if (string.IsNullOrEmpty(web.MasterUrl) == true)
            {
                Debug.Fail("web.MasterUrl is null or empty.");

                // If this actually happens in a Release build, simply log a
                // warning, but don't throw an exception
                Logger.LogWarning(
                    CultureInfo.InvariantCulture,
                    "ApplicationPageHandlerFactory - "
                    + "web.MasterUrl is null or empty for {0}.",
                    web.Url);

                return;
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Setting master page for {0} to {1}...",
                page.Request.Url,
                web.MasterUrl);

            page.MasterPageFile = web.MasterUrl;
        }

        private static bool ShouldOverrideMasterPage(
            Page page)
        {
            Debug.Assert(page != null);

            // Note: _layouts/Help.aspx does not specify a master page
            if (string.IsNullOrEmpty(page.MasterPageFile))
            {
                Logger.LogDebug("Page does not specify master page file.");
                return false;
            }

            // Note: _layouts/Error.aspx specifies "~/_layouts/simple.master"
            string masterPageFile = page.MasterPageFile.ToLower(
                CultureInfo.InvariantCulture);

            if (masterPageFile.Contains("_layouts/application.master") == false)
            {
                Logger.LogDebug("Master page file does not specify _layouts/application.master.");
                return false;
            }

            return true;
        }
    }
}
```

Note that the HttpHandler must hook into the PreInit phase of the page lifecycle, because ASP.NET only allows the master page to be changed up to this point. Also note that there's a little more conditional logic than you might expect in order to account for infrequent -- but nevertheless very important -- scenarios, such as deleting a site.

When deleting a site, `SPContext.Current` is null (or at least it was in the original RTM build of MOSS 2007 when I originally developed this feature).

Also note that some \_layouts pages do not specify application.master -- specifically the out-of-the-box help page (i.e. \_layouts/Help.aspx) and error page (i.e. \_layouts/Error.aspx). Since these master pages may not specify a master page at all, or specify a master page with different placeholders than application.master, we certainly don't want to attempt to override with application.master.

To configure the custom HttpHandler for application pages (a.k.a. \_layouts pages), modify the Web.config file in

> %ProgramFiles%\Common Files\Microsoft Shared\web server extensions\12\TEMPLATE\LAYOUTS

Simply comment out the default PageHandlerFactory and add the custom ApplicationPageHandlerFactory:

```
    <httpHandlers>
      <!--
      <add verb="*" path="*.aspx"
        type="System.Web.UI.PageHandlerFactory, System.Web,
          Version=1.0.5000.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
      -->
      <add verb="*" path="*.aspx"
        type="Fabrikam.Project1.PublishingLayouts.Web.UI.ApplicationPageHandlerFactory,
          Fabrikam.Project1.PublishingLayouts,
          Version=1.0.0.0, Culture=neutral, PublicKeyToken=d006e8e37357742f" />
    </httpHandlers>
```

Be aware that if you use the approach shown here -- specifically, setting the master page for an application page based on the current site context -- then you must use a custom master page that includes all of the placeholders included in both default.master and application.master (as noted in my [previous post](/blog/jjameson/2009/09/19/moss-2007-master-page-comparison)). Otherwise, you'll encounter an error similar to the following:

{{< blockquote "font-italic text-danger" >}}

Cannot find ContentPlaceHolder 'PlaceHolderPageDescriptionRowAttr' in the master page '/\_catalogs/masterpage/default.master', verify content control's ContentPlaceHolderID attribute in the content page.

{{< /blockquote >}}

Lastly, I want to point out that this approach only affects application pages -- not site and system pages. In other words, the code shown above in ApplicationPageHandlerFactory is not executed on every page request for your SharePoint site. It's also probably worth mentioning that in order to avoid any possibility of having Microsoft Support throw the "unsupported" trump card on you when opening a service request (i.e. SRX), you should probably temporarily revert the Web.config change above (so that the default PageHandlerFactory is used) and reproduce your problem with the default, out-of-the-box configuration.

