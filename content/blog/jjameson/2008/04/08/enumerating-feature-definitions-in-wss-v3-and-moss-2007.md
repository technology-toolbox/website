---
title: "Enumerating Feature Definitions in WSS v3 and MOSS 2007"
date: 2008-04-08T18:39:00-06:00
excerpt: "There might be occasions where you need to \"decode\" the feature GUID in Windows SharePoint Services (WSS) v3 or Microsoft Office SharePoint Server (MOSS) 2007. For these (admittedly rare) situations, I have attached an Excel spreadsheet containing the..."
aliases: ["/blog/jjameson/archive/2008/04/08/enumerating-feature-definitions-in-wss-v3-and-moss-2007.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2008/04/08/enumerating-feature-definitions-in-wss-v3-and-moss-2007.aspx](http://blogs.msdn.com/b/jjameson/archive/2008/04/08/enumerating-feature-definitions-in-wss-v3-and-moss-2007.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

There might be occasions where you need to "decode" the feature GUID in Windows
SharePoint Services (WSS) v3 or Microsoft Office SharePoint Server (MOSS) 2007.
For these (admittedly rare) situations, I have attached an Excel spreadsheet
containing the out-of-the-box feature definitions from one of my MOSS 2007
Service Pack 1 (SP1) VMs.

Note that there is no real "magic" in generating this list -- just a little code
to enumerate the `SPFeatureDefinitionCollection` on the `SPFarm` object.
However, you'll notice in the following code sample that a little effort is
required to convert the GUID for a solution into the corresponding solution name
(handling "hidden" solutions, such as Microsoft.Office.Excel.Server):

```
using System;

using Microsoft.SharePoint;
using Microsoft.SharePoint.Administration;

namespace EnumFeatureDefs
{
    class Program
    {
        private static readonly Guid excelServerSolutionId =
            new Guid("{7ed6cd55-b479-4eb7-a529-e99a24c10bd3}");

        static void Main(
            string[] args)
        {
            Uri url = new Uri("http://foobar/sites/TfsLite");

            SPWebApplication webApp = SPWebApplication.Lookup(url);

            EnumerateFeatureDefinitions(webApp.Farm.FeatureDefinitions,
                webApp.Farm.Solutions);
        }

        private static void EnumerateFeatureDefinitions(
            SPFeatureDefinitionCollection featureDefs,
            SPSolutionCollection solutions)
        {
            Console.WriteLine("{0}\t{1}\t{2}\t{3}",
                "Feature Id",
                "Display Name",
                "Scope",
                "Solution");

            foreach (SPFeatureDefinition featureDef in featureDefs)
            {
                string solutionName = string.Empty;

                if (featureDef.SolutionId != Guid.Empty)
                {
                    if (featureDef.SolutionId.Equals(excelServerSolutionId))
                    {
                        solutionName = "Microsoft.Office.Excel.Server";
                    }
                    else
                    {
                        SPSolution solution = solutions[featureDef.SolutionId];

                        if (solution == null)
                        {
                            // Similar to Microsoft.Office.Excel.Server,
                            // the solution is marked Hidden="TRUE" and
                            // therefore does not appear in the list of
                            // solutions for the farm.
                            solutionName = "(Hidden)";
                        }
                        else
                        {
                            solutionName = solution.DisplayName;
                        }
                    }
                }

                Console.WriteLine("{0}\t{1}\t{2}\t{3}",
                    featureDef.Id,
                    featureDef.DisplayName,
                    featureDef.Scope,
                    solutionName);
            }
        }
    }
}
```

If you are wondering why you would ever need this kind of information, refer to
my next post (
[Creating a Site Template in MOSS 2007 that Works in WSS v3](/blog/jjameson/2008/04/08/creating-a-site-template-in-moss-2007-that-works-in-wss-v3)).

Here is the list in case you don't want to download the attached file:

{{< table class="small" caption="MOSS 2007 Feature Definitions" >}}

| Feature Id | Display Name | Scope | Solution |
| --- | --- | --- | --- |
| 99ee0928-7342-4739-865d-35b61ea4eaf0 | BDCAdminUILinks | Farm | Microsoft.Office.Excel.Server |
| cdfa39c6-6413-4508-bccf-bf30368472b3 | DataConnectionLibraryStapling | Farm | Microsoft.Office.Excel.Server |
| e4e6a041-bc5b-45cb-beab-885a27079f74 | ExcelServer | Farm | Microsoft.Office.Excel.Server |
| a10b6aa4-135d-4598-88d1-8d4ff5691d13 | ipfsAdminLinks | Farm | Microsoft.Office.Excel.Server |
| a573867a-37ca-49dc-86b0-7d033a7ed2c8 | PremiumSiteStapling | Farm | Microsoft.Office.Excel.Server |
| 395702f0-184c-46a2-9bb5-0a64b048738c | Analytics | Farm |  |
| 97a2485f-ef4b-401f-9167-fa4fe177c6f6 | BaseSiteStapling | Farm |  |
| aeef8777-70c0-429f-8a13-f12db47a6d47 | BulkWorkflow | Farm |  |
| 0f121a23-c6bc-400f-87e4-e6bbddf6916d | ContentLightup | Farm |  |
| fead7313-4b9e-4632-80a2-ff00a2d83297 | ContentTypeSettings | Farm |  |
| 1ec2c859-e9cb-4d79-9b2b-ea8df09ede22 | DMContentTypeSettings | Farm |  |
| 81ebc0d6-8fb2-4e3f-b2f8-062640037398 | EnhancedHtmlEditing | Farm |  |
| 0125140f-7123-4657-b70a-db9aa1f209e5 | FeaturePushdown | Farm |  |
| 4d0d9bec-5103-4663-b88d-27cfab1029ff | FeaturePushdownTask | Farm |  |
| 319d8f70-eb3a-4b44-9c79-2087a87799d6 | GlobalWebParts | Farm |  |
| fc33ba3b-7919-4d7e-b791-c6aeccf8f851 | ListTargeting | Farm |  |
| 8a663fe0-9d9c-45c7-8297-66365ad50427 | MasterSiteDirectoryControl | Farm |  |
| 69cc9662-d373-47fc-9449-f18d11ff732c | MySite | Farm |  |
| 0faf7d1b-95b1-4053-b4e2-19fd5c9bbc88 | MySiteCleanup | Farm |  |
| c922c106-7d0a-4377-a668-7f13d52cb80f | OSearchCentralAdminLinks | Farm |  |
| edf48246-e4ee-4638-9eed-ef3d0aee7597 | OSearchPortalAdminLinks | Farm |  |
| fcd4c704-ed7a-42fb-ab30-2bb0ab6494c8 | OSearchSRPAdminLinks | Farm |  |
| af847aa9-beb6-41d4-8306-78e41af9ce25 | ProfileSynch | Farm |  |
| 001f4bd7-746d-403b-aa09-a6cc43de7942 | PublishingStapling | Farm |  |
| 6d127338-5e7d-4391-8f62-a11e43b1d404 | RecordsManagement | Farm |  |
| f324259d-393d-4305-aa48-36e8d9a7a0d6 | SharedServices | Farm |  |
| fead7313-4b9e-4632-80a2-98a2a2d83297 | SiteSettings | Farm |  |
| 937f97e9-d7b4-473d-af17-b03951b2c66b | SkuUpgradeLinks | Farm |  |
| 65d96c6b-649a-4169-bf1d-b96505c60375 | SlideLibraryActivation | Farm |  |
| 612d671e-f53d-4701-96da-c3a4ee00fdc5 | SpellChecking | Farm |  |
| 713a65a1-2bc7-4e62-9446-1d0b56a8bf7f | SPSDisco | Farm |  |
| 11df38ab-5bbb-4304-9da8-221c5c4100b0 | SpsSsoLinks | Farm |  |
| c43a587e-195b-4d29-aba8-ebb22b48eb1a | SRPProfileAdmin | Farm |  |
| ee21b29b-b0d0-42c6-baff-c97fd91786e6 | StapledWorkflows | Farm |  |
| 82e2ea42-39e2-4b27-8631-ed54c1cfc491 | TransMgmtFunc | Farm |  |
| f0deabbb-b0f6-46ba-8e16-ff3b44461aeb | UserMigrator | Farm |  |
| e15ed6d2-4af1-4361-89d3-2acf8cd485de | ExcelServerWebApplication | WebApplication | Microsoft.Office.Excel.Server |
| 0ea1c3b6-6ac0-44aa-9f3f-05e8dbe6d70b | PremiumWebApplication | WebApplication | Microsoft.Office.Excel.Server |
| 4f56f9fa-51a0-420c-b707-63ecbb494db1 | BaseWebApplication | WebApplication |  |
| d992aeca-3802-483a-ab40-6c9376300b61 | BulkWorkflowTimerJob | WebApplication |  |
| bc29e863-ae07-4674-bd83-2c6d0aa5623f | OSearchBasicFeature | WebApplication |  |
| 4750c984-7721-4feb-be61-c660c6190d43 | OSearchEnhancedFeature | WebApplication |  |
| 14173c38-5e2d-4887-8134-60f9df889bad | PageConverters | WebApplication |  |
| 1dbf6063-d809-45ea-9203-d3ba4a64f86d | SearchAndProcess | WebApplication |  |
| 2ac1da39-c101-475c-8601-122bc36e3d67 | SPSearchFeature | WebApplication |  |
| 43f41342-1a37-4372-8ca0-b44d881e4434 | BizAppsCTypes | Site | Microsoft.Office.Excel.Server |
| 5a979115-6b71-45a5-9881-cdc872051a69 | BizAppsFields | Site | Microsoft.Office.Excel.Server |
| 4248e21f-a816-4c88-8cab-79d82201da7b | BizAppsSiteTemplates | Site | Microsoft.Office.Excel.Server |
| 3cb475e7-4e87-45eb-a1f3-db96ad7cf313 | ExcelServerSite | Site | Microsoft.Office.Excel.Server |
| c88c4ff1-dbf5-4649-ad9f-c6c426ebcbf5 | IPFSSiteFeatures | Site | Microsoft.Office.Excel.Server |
| 8581a8a7-cf16-4770-ac54-260265ddb0b2 | PremiumSite | Site | Microsoft.Office.Excel.Server |
| b21b090c-c796-4b0f-ac0f-7ef1659c20ae | BaseSite | Site |  |
| 00bfea71-1c5e-4a24-b310-ba51c3eb7a57 | BasicWebParts | Site |  |
| 695b6570-a48b-4a8e-8ea5-26ea7fc1d162 | CTypes | Site |  |
| c85e5759-f323-4efb-b548-443d2216efb5 | ExpirationWorkflow | Site |  |
| ca7bd552-10b1-4563-85b9-5ed1d39c962a | Fields | Site |  |
| fde5d850-671e-4143-950a-87b473922dc7 | IssueTrackingWorkflow | Site |  |
| 14aafd3a-fcb9-4bb7-9ad7-d8e36b663bbd | LocalSiteDirectoryControl | Site |  |
| e978b1a6-8de7-49d0-8600-09a250354e14 | LocalSiteDirectorySettingsLink | Site |  |
| 863da2ac-3873-4930-8498-752886210911 | MySiteBlog | Site |  |
| 49571cd1-b6a1-43a3-bf75-955acc79c8d8 | MySiteHost | Site |  |
| 6928b0e5-5707-46a1-ae16-d6e52522d52b | MySiteLayouts | Site |  |
| 89e0306d-453b-4ec5-8d68-42067cdbf98e | Navigation | Site |  |
| c9c9515d-e4e2-4001-9050-74f980f93160 | OffWFCommon | Site |  |
| 7ac8cc56-d28e-41f5-ad04-d95109eb987a | OSSSearchSearchCenterUrlSiteFeature | Site |  |
| 5f3b0127-2f1d-4cfd-8dd2-85ad1fb00bfc | PortalLayouts | Site |  |
| 24d7018d-bf48-4813-a28d-dbf3dba173b1 | PublishingB2TRHop2SiteFilesUpgrade | Site |  |
| fd3dd145-e35e-4871-9a6d-bf17f28a1c19 | PublishingB2TRSiteFilesUpgrade | Site |  |
| d3f51be2-38a8-4e44-ba84-940d35be1566 | PublishingLayouts | Site |  |
| a392da98-270b-4e85-9769-04c0fde267aa | PublishingPrerequisites | Site |  |
| aebc918d-b20f-4a11-a1db-9ed84d79c87e | PublishingResources | Site |  |
| f6924d36-2fa8-4f0b-b16d-06b7250180fa | PublishingSite | Site |  |
| 8156ee99-ddfb-47bb-8835-7ae42d40d9b9 | ReportCenterCreation | Site |  |
| 7094bd89-2cfe-490a-8c7e-fbace37b4a34 | Reporting | Site |  |
| 02464c6a-9d07-4f30-ba04-e9035cf54392 | ReviewWorkflows | Site |  |
| eaf6a128-0482-4f71-9a2f-b1c650680e77 | SearchWebParts | Site |  |
| 6c09612b-46af-4b2f-8dfc-59185c962a29 | SignaturesWorkflow | Site |  |
| c6561405-ea03-40a9-a57f-f25472942a22 | TranslationWorkflow | Site |  |
| 7c637b23-06c4-472d-9a9a-7c175762c5c4 | ViewFormPagesLockDown | Site |  |
| 2ed1c45e-a73b-4779-ae81-1524e4de467a | WebPartAdderGroups | Site |  |
| d250636f-0a26-4019-8425-a5232d592c09 | AddDashboard | Web | Microsoft.Office.Excel.Server |
| 065c78be-5231-477e-a972-14177cc5b3c7 | BizAppsListTemplates | Web | Microsoft.Office.Excel.Server |
| 00bfea71-dbd7-4f72-b8cb-da7ac0440130 | DataConnectionLibrary | Web | Microsoft.Office.Excel.Server |
| 750b8e49-5213-4816-9fa2-082900c0201a | IPFSAdminWeb | Web | Microsoft.Office.Excel.Server |
| a0e5a010-1329-49d4-9e09-f280cdbed37d | IPFSWebFeatures | Web | Microsoft.Office.Excel.Server |
| 0806d127-06e6-447a-980e-2e90b03101b8 | PremiumWeb | Web | Microsoft.Office.Excel.Server |
| 2510d73f-7109-4ccc-8a1c-314894deeb3a | ReportListTemplate | Web | Microsoft.Office.Excel.Server |
| fead7313-ae6d-45dd-8260-13b563cb4c71 | AdminLinks | Web |  |
| 56dd7fe7-a155-4283-b5e6-6147560601ee | AnalyticsLinks | Web |  |
| 00bfea71-d1ce-42de-9c63-a44004ce0104 | AnnouncementsList | Web |  |
| 99fe402e-89a0-45aa-9163-85342e865dc8 | BaseWeb | Web |  |
| 3f59333f-4ce1-406d-8a97-9ecb0ff0337f | BDR | Web |  |
| 00bfea71-7e6d-4186-9ba8-c047ac750105 | ContactsList | Web |  |
| 00bfea71-de22-43b2-a848-c05709900100 | CustomList | Web |  |
| 00bfea71-f381-423d-b9d1-da7a54c50110 | DataSourceLibrary | Web |  |
| ca2543e6-29a1-40c1-bba9-bd8510a4c17b | DeploymentLinks | Web |  |
| 00bfea71-6a49-43fa-b535-d15c05500108 | DiscussionsList | Web |  |
| 00bfea71-e717-4e80-aa17-d0c71b360101 | DocumentLibrary | Web |  |
| 00bfea71-ec85-4903-972d-ebe475780106 | EventsList | Web |  |
| 00bfea71-513d-4ca0-96c2-6a47775c0119 | GanttTasksList | Web |  |
| 5b1e6e3b-83c2-483b-8500-16a025777ed1 | GradualUpgrade | Web |  |
| 00bfea71-3a1d-41d3-a0ee-651d11570120 | GridList | Web |  |
| 9e56487c-795a-4077-9425-54a1ecb84282 | Hold | Web |  |
| 00bfea71-5932-4f9c-ad71-1557e5751100 | IssuesList | Web |  |
| 6e53dd27-98f2-4ae5-85a0-e9a8ef4aa6df | LegacyDocumentLibrary | Web |  |
| 00bfea71-2062-426c-90bf-714c59600103 | LinksList | Web |  |
| 8f15b342-80b1-4508-8641-0751e2b55ca6 | LocalSiteDirectoryMetaData | Web |  |
| 7fe16263-b3fd-454f-a3e8-ed05fdf2adb6 | MigrationLinks | Web |  |
| f41cc668-37e5-4743-b4a8-74d1db3fd8a4 | MobilityRedirect | Web |  |
| 6adff05c-d581-4c05-a6b9-920f15ec6fd9 | MySiteNavigation | Web |  |
| 034947cc-c424-47cd-a8d1-6014f0e36925 | MySiteQuickLaunch | Web |  |
| 541f5f57-c847-4e16-b59a-b31e90e6f9ea | NavigationProperties | Web |  |
| 00bfea71-f600-43f6-a895-40c0de7b0117 | NoCodeWorkflowLibrary | Web |  |
| 068f8656-bea6-4d60-a5fa-7f077f8f5c20 | OsrvLinks | Web |  |
| 0b4aad40-406f-425c-bdd9-5894c42cffcb | OsrvTasks | Web |  |
| 10bdac29-a21a-47d9-9dff-90c7cae1301e | OssNavigation | Web |  |
| 7acfcb9d-8e8f-4979-af7e-8aed7e95245e | OSSSearchSearchCenterUrlFeature | Web |  |
| 00bfea71-52d4-45b3-b544-b1c71b620109 | PictureLibrary | Web |  |
| 22a9ef51-737b-4ff2-9346-694633fe4416 | Publishing | Web |  |
| 94c94ca6-b32f-4da9-a9e3-1f3d343d7ecb | PublishingWeb | Web |  |
| 306936fd-9806-4478-80d1-7e397bfa6474 | RedirectPageContentTypeBinding | Web |  |
| e8734bb6-be8e-48a1-b036-5a40ff0b8a81 | RelatedLinksScopeSettingsLink | Web |  |
| c5d947d6-b0a2-4e07-9929-8e54f5a9fff9 | ReportCenterSampleData | Web |  |
| a311bf68-c990-4da3-89b3-88989a3d7721 | SitesList | Web |  |
| 0be49fe9-9bc9-409d-abf9-702753bd878d | SlideLibrary | Web |  |
| 00bfea71-eb8a-40b1-80c7-506be7590102 | SurveysList | Web |  |
| 00bfea71-a83e-497e-9ba0-7a5c597d0107 | TasksList | Web |  |
| 00bfea71-4ea5-48d4-a4ad-7ea5c011abe5 | TeamCollab | Web |  |
| 29d85c25-170c-4df9-a641-12db0b9d4130 | TransMgmtLib | Web |  |
| 2fa4db13-4109-4a1d-b47c-c7991d4cc934 | UpgradeOnlyFile | Web |  |
| 00bfea71-c796-4402-9f2f-0eb9a6e71b18 | WebPageLibrary | Web |  |
| 8c6a6980-c3d9-440e-944c-77f93bc65a7e | WikiWelcome | Web |  |
| 00bfea71-4ea5-48d4-a4ad-305cf7030140 | WorkflowHistoryList | Web |  |
| 00bfea71-2d77-4a75-9fca-76516689e21a | workflowProcessList | Web |  |
| 00bfea71-1e1d-4562-b56a-f05371bb0115 | XmlFormLibrary | Web |  |

{{< /table >}}

