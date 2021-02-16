---
title: "Creating a Site Template in MOSS 2007 that Works in WSS v3"
date: 2008-04-08T14:35:00-07:00
excerpt: "Shortly after publishing my previous post covering \"TFS Lite\" for WSS v3 , Dragan Panjkov noted that attempting to create a new site in WSS v3 using the site template that I originally provided resulted in the following error: 
 The template you have..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/04/08/creating-a-site-template-in-moss-2007-that-works-in-wss-v3.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/04/08/creating-a-site-template-in-moss-2007-that-works-in-wss-v3.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

Shortly after publishing my previous post covering ["TFS Lite"
for WSS v3](/blog/jjameson/2008/04/07/tfs-lite-for-wss-v3), Dragan Panjkov noted that attempting to create a new site in WSS  v3 using the site template that I originally provided resulted in the following  error:

{{< blockquote "font-italic text-danger" >}}

The template you have chosen is invalid or cannot be found.

{{< /blockquote >}}

Ouch.

Okay, time to fess up. You see, I didn't really create the template using WSS  v3. Rather, I created it on one of my MOSS 2007 VMs and simply *assumed*  it would work with WSS v3. What is it they say about assumptions again? Never mind,  don't answer that.

To investigate the issue, I "spun up" a new VM based on my SysPrep'ed image of  Windows Server 2003 SP2, joined the new VM to my domain, ran {{< kbd "wuauclt /detectnow" >}}  a couple of times to install the 40 or so hotfixes that have been released for Windows  Server 2003 since SP2, and then installed WSS with SP1. Sure enough, about an hour  after discovering the error reported by Dragan, I was able to repro the problem.  [Thank goodness for Virtual Server, SysPrep, slipstreamed service packs, and a local  intranet location of Windows Update!]

Digging into the SharePoint logs I noticed the following:

```
04/08/2008 ... Applying template "TfsLite.stp" to web at URL "http://wss-dev/Test".	 
04/08/2008 ... Failed to get the site template for language 1033, search key 'TfsLite.stp'. This warning is expected when provisioning from a custom web template.
04/08/2008 ... Marking web-scoped features active from manifest at URL "http://wss-dev/Test"
04/08/2008 ... Failed to mark site-scoped features active in site 'http://wss-dev/Test'.
04/08/2008 ... Failed to apply template "TfsLite.stp" to web at URL "http://wss-dev/Test".
04/08/2008 ... Failed to apply template "TfsLite.stp" to web at URL "http://wss-dev/Test", error The template you have chosen is invalid or cannot be found. 0x81071e44
04/08/2008 ... The template you have chosen is invalid or cannot be found.
```

At this point, the problem seemed fairly obvious: the error occurred while trying  to activate site-scoped features on the new site. Well, duh, that makes sense. I  must have had one or more MOSS 2007-specific features enabled on the site collection  that I used when originally creating the site template. Since these features are  not available in WSS v3, an error occurs.

I then deactivated all of the following features on the site collection and saved  the site as a template again:

- Collect Signatures Workflow
- Disposition Approval Workflow
- Office SharePoint Server Enterprise Site Collection features
- Office SharePoint Server Standard Site Collection features
- Reporting
- Routing Workflows
- Three-state workflow
- Translation Management Workflow

Unfortunately, after copying the updated STP file from my MOSS 2007 VM to my  WSS v3 VM, I encountered the same problem when trying to create a site from the  template. Ugh.

Convinced that the problem was still due to a feature specific to MOSS 2007 (i.e.  not available in WSS v3), I then created a "vanilla" Team Site template using the  WSS v3 VM and subsequently compared the manifest files from the two templates to  see if I could identify which feature was breaking the site creation process. [If  you are not aware of the structure of an STP file, it is essentially a CAB file  with an embedded XML manifest file. All you need to do is rename the STP file to  have a .cab file extension and then extract the manifest.xml to examine the contents.]

Using WinDiff to compare the manifest file from the TFS Lite site template with  the "vanilla" Team Site template, I observed the following differences:

```
<SiteFeatures>
  <!-- CTypes -->
  <Feature ID="695b6570-a48b-4a8e-8ea5-26ea7fc1d162" />
  <!-- Fields -->
  <Feature ID="ca7bd552-10b1-4563-85b9-5ed1d39c962a" />
  <!-- IssueTrackingWorkflow -->
  <Feature ID="fde5d850-671e-4143-950a-87b473922dc7" />
  <!-- BasicWebParts -->
  <Feature ID="00bfea71-1c5e-4a24-b310-ba51c3eb7a57" />
  <!-- IPFSSiteFeatures -->
  <Feature ID="c88c4ff1-dbf5-4649-ad9f-c6c426ebcbf5" />
  <!-- ExcelServerSite -->
  <Feature ID="3cb475e7-4e87-45eb-a1f3-db96ad7cf313" />
</SiteFeatures>
<WebFeatures>
  <!-- WebPageLibrary -->
  <Feature ID="00bfea71-c796-4402-9f2f-0eb9a6e71b18" />
  <!-- TransMgmtLib -->
  <Feature ID="29d85c25-170c-4df9-a641-12db0b9d4130" />
  <!-- BizAppsListTemplates -->
  <Feature ID="065c78be-5231-477e-a972-14177cc5b3c7" />
  <!-- IssuesList -->
  <Feature ID="00bfea71-5932-4f9c-ad71-1557e5751100" />
  <!-- PremiumWeb -->
  <Feature ID="0806d127-06e6-447a-980e-2e90b03101b8" />
  <!-- WorkflowHistoryList -->
  <Feature ID="00bfea71-4ea5-48d4-a4ad-305cf7030140" />
  <!-- ReportListTemplate -->
  <Feature ID="2510d73f-7109-4ccc-8a1c-314894deeb3a" />
  <!-- NoCodeWorkflowLibrary -->
  <Feature ID="00bfea71-f600-43f6-a895-40c0de7b0117" />
  <!-- SurveysList -->
  <Feature ID="00bfea71-eb8a-40b1-80c7-506be7590102" />
  <!-- RelatedLinksScopeSettingsLink -->
  <Feature ID="e8734bb6-be8e-48a1-b036-5a40ff0b8a81" />
  <!-- AnalyticsLinks -->
  <Feature ID="56dd7fe7-a155-4283-b5e6-6147560601ee" />
  <!-- GridList -->
  <Feature ID="00bfea71-3a1d-41d3-a0ee-651d11570120" />
  <!-- GanttTasksList -->
  <Feature ID="00bfea71-513d-4ca0-96c2-6a47775c0119" />
  <!-- SlideLibrary -->
  <Feature ID="0be49fe9-9bc9-409d-abf9-702753bd878d" />
  <!-- LinksList -->
  <Feature ID="00bfea71-2062-426c-90bf-714c59600103" />
  <!-- MobilityRedirect -->
  <Feature ID="f41cc668-37e5-4743-b4a8-74d1db3fd8a4" />
  <!-- workflowProcessList -->
  <Feature ID="00bfea71-2d77-4a75-9fca-76516689e21a" />
  <!-- TasksList -->
  <Feature ID="00bfea71-a83e-497e-9ba0-7a5c597d0107" />
  <!-- TeamCollab -->
  <Feature ID="00bfea71-4ea5-48d4-a4ad-7ea5c011abe5" />
  <!-- BaseWeb -->
  <Feature ID="99fe402e-89a0-45aa-9163-85342e865dc8" />
  <!-- AnnouncementsList -->
  <Feature ID="00bfea71-d1ce-42de-9c63-a44004ce0104" />
  <!-- PictureLibrary -->
  <Feature ID="00bfea71-52d4-45b3-b544-b1c71b620109" />
  <!-- ContactsList -->
  <Feature ID="00bfea71-7e6d-4186-9ba8-c047ac750105" />
  <!-- CustomList -->
  <Feature ID="00bfea71-de22-43b2-a848-c05709900100" />
  <!-- DocumentLibrary -->
  <Feature ID="00bfea71-e717-4e80-aa17-d0c71b360101" />
  <!-- DiscussionsList -->
  <Feature ID="00bfea71-6a49-43fa-b535-d15c05500108" />
  <!-- DataSourceLibrary -->
  <Feature ID="00bfea71-f381-423d-b9d1-da7a54c50110" />
  <!-- DataConnectionLibrary -->
  <Feature ID="00bfea71-dbd7-4f72-b8cb-da7ac0440130" />
  <!-- EventsList -->
  <Feature ID="00bfea71-ec85-4903-972d-ebe475780106" />
  <!-- XmlFormLibrary -->
  <Feature ID="00bfea71-1e1d-4562-b56a-f05371bb0115" />
  <!-- IPFSWebFeatures -->
  <Feature ID="a0e5a010-1329-49d4-9e09-f280cdbed37d" />
</WebFeatures>
```

Note that I've annotated the features with comments noting the feature name (using  the spreadsheet provided in [my previous post](/blog/jjameson/2008/04/08/enumerating-feature-definitions-in-wss-v3-and-moss-2007)) and highlighted the differences in a similar fashion to WinDiff  -- the "left-only" features (i.e. those only found  in the "vanilla" Team Site template created in WSS v3) in red and the "right-only" features (i.e. those only found  in the TFS Lite site template created in MOSS 2007) in yellow.

Note that simply deactivating all of the site collection features in MOSS 2007  does not make the corresponding site template compatible with WSS v3. However, once  I modified the manifest.xml to remove the MOSS 2007-specific features, recreated  the CAB file, and changed the file extension back to .stp, I was able to successfully  create a site in WSS v3 based on the site template. Woohoo!

