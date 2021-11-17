---
title: Enumerating Feature Definitions in WSS v3 and MOSS 2007
date: 2008-04-08T18:39:00-06:00
description:
  'There might be occasions where you need to "decode" the feature GUID in
  Windows SharePoint Services (WSS) v3 or Microsoft Office SharePoint Server
  (MOSS) 2007. For these (admittedly rare) situations, I have attached an Excel
  spreadsheet containing the...'
aliases:
  [
    "/blog/jjameson/archive/2008/04/08/enumerating-feature-definitions-in-wss-v3-and-moss-2007.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2008/04/08/enumerating-feature-definitions-in-wss-v3-and-moss-2007.aspx"
attachment:
  url: "https://assets.technologytoolbox.com/blog/jjameson/Documents/MOSS 2007 Feature Definitions.xlsx"
  fileName: MOSS 2007 Feature Definitions.xlsx
  fileSizeInBytes: 24852
---

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

```C#
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
my next post
([Creating a Site Template in MOSS 2007 that Works in WSS v3](/blog/jjameson/2008/04/08/creating-a-site-template-in-moss-2007-that-works-in-wss-v3)).

Here is the list in case you don't want to download the attached file:

<div class="d-md-none">
    <a href='{{< relref "resources/table-1-popout" >}}' target="_blank">Table 1 - MOSS 2007 Feature Definitions</a>
    {{< svg-icon "arrow-up-right-square" >}}
    <p>(Insufficient width to show table content here.)</p>
</div>
<div class="d-none d-md-block">
    {{< include-html "resources/table-1.html" >}}
</div>
