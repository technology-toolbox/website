---
title: "AJAX in MOSS 2007 -- The Easy Way, Part 1"
date: 2010-03-23T16:42:00-06:00
excerpt: "In my previous post , I showed how you can quickly create a Web application in Microsoft Office SharePoint Server (MOSS) 2007 and configure it for anonymous access and Forms-Based Authentication. 
 Let's suppose that instead of configuring FBA and anonymous..."
aliases: ["/blog/jjameson/archive/2010/03/23/ajax-in-moss-2007-the-easy-way-part-1.aspx"]
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/03/23/ajax-in-moss-2007-the-easy-way-part-1.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/03/23/ajax-in-moss-2007-the-easy-way-part-1.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In my
[previous post](/blog/jjameson/2010/03/23/forms-based-authentication-in-moss-2007-the-easy-way),
I showed how you can quickly create a Web application in Microsoft Office
SharePoint Server (MOSS) 2007 and configure it for anonymous access and
Forms-Based Authentication.

Let's suppose that instead of configuring FBA and anonymous access, you want to
configure AJAX on your Web application instead.

Similar to the **Fabrikam.Demo.Web.FormsBasedAuthenticationConfiguration**
feature that I shared in the previous post, the
**Fabrikam.Demo.Web.AjaxConfiguration** feature adds the necessary Web.config
elements to support AJAX on a SharePoint site.

Mike Ammerlaan's post from a couple of years ago introduced the concept of
[integrating ASP.NET AJAX with SharePoint](http://sharepoint.microsoft.com/blogs/mike/Lists/Posts/Post.aspx?ID=3).
The detailed configuration steps eventually made their way onto MSDN as well:

{{< reference
title="Installing ASP.NET 2.0 AJAX Extensions 1.0 in Windows SharePoint Services Version 3.0"
linkHref="http://msdn.microsoft.com/en-us/library/bb861898.aspx" >}}

However, if you look at the comments on the previous MSDN article, it appears
that the prescriptive guidance isn't always easy to follow and implement. I'll
be the first to admit that copying and pasting lots of "configuration goo" can
be problematic.

Using my custom **
[SharePointWebConfigHelper](/blog/jjameson/2010/03/23/introducing-the-sharepointwebconfighelper-class)**
class, it's pretty easy to add the slew of Web.config modifications that are
required to get ASP.NET AJAX working on a SharePoint site.

Since I didn't necessarily want to limit this configuration to a SharePoint
feature, I placed the bulk of the code in the **SharePointAjaxHelper** class.
Consequently, enabling AJAX is simply a matter of calling the following method:

```
    SharePointAjaxHelper.AddAjaxWebConfigModifications(webApp);
```

Likewise, disabling AJAX is simply a matter of calling the following method:

```
    SharePointAjaxHelper.RemoveAjaxWebConfigModifications(webApp);
```

Note that due to the bug in the **SPWebConfigModification** infrastructure that
I've mentioned before, SharePoint only removes the modifications from the
Web.config file for the default zone (not, for example, the Internet zone).
However, it's probably not a big deal in this particular case because what's the
likelihood that you will start using AJAX and then later on decide to stop using
it? Probably "next to zilch" would be my guess.

Since it's just one line of code, I don't even bother with a separate **
[FeatureConfigurator](/blog/jjameson/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator)**
class for the **Fabrikam.Demo.Web.AjaxConfiguration** feature. Instead, I just
stuff the code directly into the class that inherits from **
[SPFeatureReceiver](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spfeaturereceiver.aspx)**:

```
using System;
using System.Security.Permissions;

using Microsoft.SharePoint;
using Microsoft.SharePoint.Administration;
using Microsoft.SharePoint.Security;

using Fabrikam.Demo.CoreServices.SharePoint;

namespace Fabrikam.Demo.Web.AjaxConfiguration
{
    /// <summary>
    /// Default feature receiver to trap the installation, activation,
    /// deactivation, and uninstallation of the
    /// <b>Fabrikam.Demo.Web.AjaxConfiguration</b> feature.
    /// </summary>
    [CLSCompliant(false)]
    public class DefaultFeatureReceiver : SPFeatureReceiver
    {
        /// <summary>
        /// Occurs after the Feature is activated.
        /// </summary>
        /// <param name="properties">An
        /// <see cref="Microsoft.SharePoint.SPFeatureReceiverProperties" />
        /// object that represents the properties of the event.</param>
        [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
        public override void FeatureActivated(
            SPFeatureReceiverProperties properties)
        {
            if (properties == null)
            {
                throw new ArgumentNullException("properties");
            }

            SPWebApplication webApp = (SPWebApplication)properties.Feature.Parent;
            SharePointAjaxHelper.AddAjaxWebConfigModifications(webApp);
        }

        /// <summary>
        /// Occurs after the Feature is deactivated.
        /// </summary>
        /// <param name="properties">An
        /// <see cref="Microsoft.SharePoint.SPFeatureReceiverProperties" />
        /// object that represents the properties of the event.</param>
        [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
        public override void FeatureDeactivating(
            SPFeatureReceiverProperties properties)
        {
            if (properties == null)
            {
                throw new ArgumentNullException("properties");
            }

            SPWebApplication webApp = (SPWebApplication)properties.Feature.Parent;
            SharePointAjaxHelper.RemoveAjaxWebConfigModifications(webApp);
        }

        /// <summary>
        /// Occurs after the Feature is installed.
        /// </summary>
        /// <param name="properties">An
        /// <see cref="Microsoft.SharePoint.SPFeatureReceiverProperties" />
        /// object that represents the properties of the event.</param>
        [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
        public override void FeatureInstalled(
            SPFeatureReceiverProperties properties)
        {
        }

        /// <summary>
        /// Occurs after the Feature is uninstalled.
        /// </summary>
        /// <param name="properties">An
        /// <see cref="Microsoft.SharePoint.SPFeatureReceiverProperties" />
        /// object that represents the properties of the event.</param>
        [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
        public override void FeatureUninstalling(
            SPFeatureReceiverProperties properties)
        {
        }
    }
}
```

I also created custom STSADM commands to enable and disable AJAX. For example,
after deploying the **Fabrikam.Demo.StsAdm.Commands.wsp**, the following command
can be used to enable AJAX:

{{< console-block-start >}}

{{< kbd "stsadm -o fabrikam-enableajax" >}}

```
	Adds the necessary configuration changes to enable AJAX on a web application.

       -url <url of the Web application to enable AJAX on>
```

{{< console-block-end >}}

Note that since the **SPWebConfigModification** class adds the changes to the
Web.config file for each zone, there's no need to run this command multiple
times with different URLs for the same Web application (e.g. default zone and
Internet zone). You can just run it once and "forget about it."

If you download the attached code, build it, and run the following commands, you
can subsequently add the sample AJAX Web Part (**Fabrikam Sample AJAX Update**)
to the home page:

```
set FABRIKAM_DEMO_URL=http://fabrikam-local
set FABRIKAM_BUILD_CONFIGURATION=Debug
set FABRIKAM_DEMO_APP_POOL_PASSWORD={some password}
cd \NotBackedUp\Fabrikam\Demo\Main\Source\StsAdm\Commands\DeploymentFiles\Scripts
"Add Solution.cmd"
"Deploy Solution.cmd"
cd ..\..\..\..\DeploymentFiles\Scripts
"Create Web Applications.cmd"
cd ..\..\Web\DeploymentFiles\Scripts
"Add Solution.cmd"
"Deploy Solution.cmd"
"Activate Features.cmd"
```

> **Important**
>
> There is a bug in several of the out-of-the-box master pages in MOSS 2007
> (including **BlueBand.master**) that prevent AJAX from working correctly. The
> asynchronous postback is triggered and processed on the server, but the
> response is never reflected in the corresponding **UpdatePanel**. The problem
> is that the `<WebPartPages:SPWebPartManager>` is declared outside of the
> `<form>` element.
>
> To fix this, you can edit the master page (e.g. using SharePoint Designer) and
> move `<WebPartPages:SPWebPartManager>` inside the `<form>` element -- or
> simply change the site to use **default.master**.
>
> Yvan Duhamel has more detail in
> [his post about this issue](http://blogs.msdn.com/yvan_duhamel/archive/2009/05/19/ajax-postbacks-not-working-with-any-masterpage-other-than-default-master.aspx),
> if you are interested.

The following screenshot shows what the site looks like after creating the Web
application, fixing the master page, and adding the sample Web Part. Note that
this sample isn't configured for FBA and anonymous access (since I was trying to
pare down the samples as much as possible).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/AJAX-in-SharePoint-600x417.png"
alt="AJAX in SharePoint" class="screenshot" height="417" width="600"
title="Figure 1: AJAX in SharePoint" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/AJAX-in-SharePoint-989x688.png)

Here's the current implementation of **SharePointAjaxHelper** (in case you don't
want to bother downloading the attachment in order to view the code):

```
using System;
using System.Diagnostics;
using System.Globalization;

using Microsoft.SharePoint.Administration;

using Fabrikam.Demo.CoreServices.Logging;

namespace Fabrikam.Demo.CoreServices.SharePoint
{
    /// <summary>
    /// Exposes static methods for configuring AJAX on SharePoint Web
    /// applications. This class cannot be inherited.
    /// </summary>
    /// <remarks>
    /// All methods of the <c>SharePointAjaxHelper</c> class are static and
    /// can therefore be called without creating an instance of the class.
    /// </remarks>
    [CLSCompliant(false)]
    public static class SharePointAjaxHelper
    {
        private const string ajaxWebConfigModificationOwner =
"Fabrikam.Demo.CoreServices.SharePoint.SharePointAjaxHelper";

        /// <summary>
        /// Adds the necessary configuration changes to enable AJAX on the
        /// specified web application.
        /// </summary>
        /// <param name="webApp">An
        /// <see cref="Microsoft.SharePoint.Administration.SPWebApplication"/>
        /// object representing the Web application.</param>
        public static void AddAjaxWebConfigModifications(
            SPWebApplication webApp)
        {
            if (webApp == null)
            {
                throw new ArgumentNullException("webApp");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Adding AJAX Web.config modifications to Web application"
                    + " ({0})...",
                webApp.DisplayName);

            AddCoreAjaxWebConfigModifications(webApp);
            AddAssemblyWebConfigModifications(webApp);
            AddSystemWebPagesWebConfigModifications(webApp);
            AddHttpHandlerWebConfigModifications(webApp);
            AddHttpModuleWebConfigModifications(webApp);
            AddAssemblyBindingRedirectWebConfigModifications(webApp);

            webApp.Update();
            SharePointWebConfigHelper.ApplyWebConfigModifications(webApp);

            Logger.LogInfo(
                CultureInfo.InvariantCulture,
                "Successfully added AJAX Web.config modifications to Web"
                    + " application ({0}).",
                webApp.DisplayName);
        }

        /// <summary>
        /// Removes the AJAX configuration changes from the specified Web
        /// application.
        /// </summary>
        /// <param name="webApp">An
        /// <see cref="Microsoft.SharePoint.Administration.SPWebApplication"/>
        /// object representing the Web application.</param>
        public static void RemoveAjaxWebConfigModifications(
            SPWebApplication webApp)
        {
            if (webApp == null)
            {
                throw new ArgumentNullException("webApp");
            }

            Logger.LogDebug(
                CultureInfo.InvariantCulture,
                "Removing AJAX configuration from Web application ({0})...",
                webApp.DisplayName);

            SharePointWebConfigHelper.RemoveWebConfigModifications(
                webApp,
                ajaxWebConfigModificationOwner);

            Logger.LogInfo(
                CultureInfo.InvariantCulture,
                "Successfully removed AJAX configuration on Web application"
                    + " ({0}).",
                webApp.DisplayName);
        }

        private static void AddAssemblyBindingRedirectWebConfigModifications(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "*[local-name()='dependentAssembly']"
                    + "[*[local-name()='assemblyIdentity']"
                        + "/@name='System.Web.Extensions']",
                "configuration/runtime"
                    + "/*[local-name()='assemblyBinding'"
                        + " and namespace-uri()"
                            + "='urn:schemas-microsoft-com:asm.v1']",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<dependentAssembly>"
                    + "<assemblyIdentity name='System.Web.Extensions'"
                        + " publicKeyToken='31bf3856ad364e35'/>"
                    + "<bindingRedirect"
                        + " oldVersion='1.0.0.0-1.1.0.0'"
                        + " newVersion='3.5.0.0'/>"
                + "</dependentAssembly>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "*[local-name()='dependentAssembly']"
                    + "[*[local-name()='assemblyIdentity']"
                        + "/@name='System.Web.Extensions.Design']",
                "configuration/runtime"
                    + "/*[local-name()='assemblyBinding'"
                        + " and namespace-uri()"
                            + "='urn:schemas-microsoft-com:asm.v1']",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<dependentAssembly>"
                    + "<assemblyIdentity"
                        + " name='System.Web.Extensions.Design'"
                        + " publicKeyToken='31bf3856ad364e35'/>"
                    + "<bindingRedirect"
                        + " oldVersion='1.0.0.0-1.1.0.0'"
                        + " newVersion='3.5.0.0'/>"
                + "</dependentAssembly>");

        }

        private static void AddAssemblyWebConfigModifications(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add["
                    + "@assembly='System.Core,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=B77A5C561934E089']",
                "configuration/system.web/compilation/assemblies",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add"
                    + " assembly='System.Core,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=B77A5C561934E089'/>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add["
                    + "@assembly='System.Data.DataSetExtensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=B77A5C561934E089']",
                "configuration/system.web/compilation/assemblies",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add"
                    + " assembly='System.Data.DataSetExtensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=B77A5C561934E089'/>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add["
                    + "@assembly='System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35']",
                "configuration/system.web/compilation/assemblies",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add"
                    + " assembly='System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35'/>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add["
                    + "@assembly='System.Xml.Linq,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=B77A5C561934E089']",
                "configuration/system.web/compilation/assemblies",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add"
                    + " assembly='System.Xml.Linq,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=B77A5C561934E089'/>");
        }

        private static void AddCoreAjaxWebConfigModifications(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "sectionGroup[@name='system.web.extensions']",
                "configuration/configSections",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
"<sectionGroup name='system.web.extensions'"
    + " type='System.Web.Configuration.SystemWebExtensionsSectionGroup,"
        + " System.Web.Extensions,"
        + " Version=3.5.0.0,"
        + " Culture=neutral,"
        + " PublicKeyToken=31BF3856AD364E35'>"
    + "<sectionGroup name='scripting'"
        + " type='System.Web.Configuration.ScriptingSectionGroup,"
            + " System.Web.Extensions,"
            + " Version=3.5.0.0,"
            + " Culture=neutral,"
            + " PublicKeyToken=31BF3856AD364E35'>"
        + "<section name='scriptResourceHandler'"
            + " type='System.Web.Configuration"
                    + ".ScriptingScriptResourceHandlerSection,"
                + " System.Web.Extensions,"
                + " Version=3.5.0.0,"
                + " Culture=neutral,"
                + " PublicKeyToken=31BF3856AD364E35'"
            + " requirePermission='false'"
            + " allowDefinition='MachineToApplication'/>"
        + "<sectionGroup name='webServices'"
            + " type='System.Web.Configuration"
                    + ".ScriptingWebServicesSectionGroup,"
                + " System.Web.Extensions,"
                + " Version=3.5.0.0,"
                + " Culture=neutral,"
                + " PublicKeyToken=31BF3856AD364E35'>"
            + "<section name='jsonSerialization'"
                + " type='System.Web.Configuration"
                        + ".ScriptingJsonSerializationSection,"
                    + " System.Web.Extensions,"
                    + " Version=3.5.0.0,"
                    + " Culture=neutral,"
                    + " PublicKeyToken=31BF3856AD364E35'"
                + " requirePermission='false'"
                + " allowDefinition='Everywhere'/>"
            + "<section name='profileService'"
                + " type='System.Web.Configuration"
                        + ".ScriptingProfileServiceSection,"
                    + " System.Web.Extensions,"
                    + " Version=3.5.0.0,"
                    + " Culture=neutral,"
                    + " PublicKeyToken=31BF3856AD364E35'"
                + " requirePermission='false'"
                + " allowDefinition='MachineToApplication'/>"
            + "<section name='authenticationService'"
                + " type='System.Web.Configuration"
                        + ".ScriptingAuthenticationServiceSection,"
                    + " System.Web.Extensions,"
                    + " Version=3.5.0.0,"
                    + " Culture=neutral,"
                    + " PublicKeyToken=31BF3856AD364E35'"
                + " requirePermission='false'"
                + " allowDefinition='MachineToApplication'/>"
            + "<section name='roleService'"
                + " type='System.Web.Configuration"
                        + ".ScriptingRoleServiceSection,"
                    + " System.Web.Extensions,"
                    + " Version=3.5.0.0,"
                    + " Culture=neutral,"
                    + " PublicKeyToken=31BF3856AD364E35'"
                + " requirePermission='false'"
                + " allowDefinition='MachineToApplication'/>"
        + "</sectionGroup>"
    + "</sectionGroup>"
+ "</sectionGroup>");

            // The system.webServer section is required for running ASP.NET
            // AJAX under Internet Information Services 7.0.  It is not
            // necessary for previous version of IIS.
            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "system.webServer",
                "configuration",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
"<system.webServer>"
    + "<validation"
        + " validateIntegratedModeConfiguration='false'/>"
    + "<modules>"
        + "<remove name='ScriptModule'/>"
        + "<add name='ScriptModule'"
            + " preCondition='managedHandler'"
            + " type='System.Web.Handlers.ScriptModule,"
                + " System.Web.Extensions,"
                + " Version=3.5.0.0,"
                + " Culture=neutral,"
                + " PublicKeyToken=31BF3856AD364E35'/>"
    + "</modules>"
    + "<handlers>"
        + "<remove name='WebServiceHandlerFactory-Integrated'/>"
        + "<remove name='ScriptHandlerFactory'/>"
        + "<remove name='ScriptHandlerFactoryAppServices'/>"
        + "<remove name='ScriptResource'/>"
        + "<add name='ScriptHandlerFactory' verb='*'"
            + " path='*.asmx' preCondition='integratedMode'"
            + " type='System.Web.Script.Services.ScriptHandlerFactory,"
                + " System.Web.Extensions,"
                + " Version=3.5.0.0,"
                + " Culture=neutral,"
                + " PublicKeyToken=31BF3856AD364E35'/>"
        + "<add name='ScriptHandlerFactoryAppServices'"
            + " verb='*' path='*_AppService.axd'"
            + " preCondition='integratedMode'"
            + " type='System.Web.Script.Services.ScriptHandlerFactory,"
                + " System.Web.Extensions,"
                + " Version=3.5.0.0,"
                + " Culture=neutral,"
                + " PublicKeyToken=31BF3856AD364E35'/>"
        + "<add name='ScriptResource'"
            + " preCondition='integratedMode'"
            + " verb='GET,HEAD' path='ScriptResource.axd'"
            + " type='System.Web.Handlers.ScriptResourceHandler,"
                + " System.Web.Extensions,"
                + " Version=3.5.0.0,"
                + " Culture=neutral,"
                + " PublicKeyToken=31BF3856AD364E35'/>"
    + "</handlers>"
+ "</system.webServer>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "SafeControl["
                    + "@Assembly='System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35']",
                "configuration/SharePoint/SafeControls",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<SafeControl"
                    + " Assembly='System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35'"
                        + " Namespace='System.Web.UI'"
                    + " TypeName='*' Safe='True'/>");
        }

        private static void AddHttpHandlerWebConfigModifications(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add[@path='*.asmx']",
                "configuration/system.web/httpHandlers",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add verb='*' path='*.asmx'"
                    + " validate='false'"
                    + " type='System.Web.Script.Services.ScriptHandlerFactory,"
                        + " System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35'/>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add[@path='*_AppService.axd']",
                "configuration/system.web/httpHandlers",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add verb='*' path='*_AppService.axd'"
                    + " validate='false'"
                    + " type='System.Web.Script.Services.ScriptHandlerFactory,"
                        + " System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35'/>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add[@path='ScriptResource.axd']",
                "configuration/system.web/httpHandlers",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add verb='GET,HEAD'"
                    + " path='ScriptResource.axd'"
                    + " type='System.Web.Handlers.ScriptResourceHandler,"
                        + " System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35'"
                        + " validate='false'/>");
        }

        private static void AddHttpModuleWebConfigModifications(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add[@name='ScriptModule']",
                "configuration/system.web/httpModules",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add name='ScriptModule'"
                    + " type='System.Web.Handlers.ScriptModule,"
                        + " System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35'/>");

        }

        private static void AddSystemWebPagesWebConfigModifications(
            SPWebApplication webApp)
        {
            Debug.Assert(webApp != null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "controls",
                "configuration/system.web/pages",
SPWebConfigModification.SPWebConfigModificationType.EnsureSection,
                null);

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add[@namespace='System.Web.UI']",
                "configuration/system.web/pages/controls",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add tagPrefix='asp'"
                    + " namespace='System.Web.UI'"
                    + " assembly='System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35'/>");

            SharePointWebConfigHelper.AddWebConfigModification(
                webApp,
                ajaxWebConfigModificationOwner,
                "add[@namespace='System.Web.UI.WebControls']",
                "configuration/system.web/pages/controls",
SPWebConfigModification.SPWebConfigModificationType.EnsureChildNode,
                "<add tagPrefix='asp'"
                    + " namespace='System.Web.UI.WebControls'"
                    + " assembly='System.Web.Extensions,"
                        + " Version=3.5.0.0,"
                        + " Culture=neutral,"
                        + " PublicKeyToken=31BF3856AD364E35'/>");

        }
    }
}
```

In [part 2](/blog/jjameson/2010/03/24/ajax-in-moss-2007-the-easy-way-part-2) of
this post, I'll cover more detail on developing AJAX Web Parts for MOSS 2007.

