---
title: "Importing Pages into MOSS 2007 from an Excel File"
date: 2009-10-08T02:54:00+08:00
excerpt: "In my previous post , I briefly introduced the concept of a utility to import pages into Microsoft Office SharePoint Server (MOSS) 2007 from an Excel input file. This can be very useful for Development and Test environments (where you frequently rebuild..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/08/importing-pages-into-moss-2007-from-an-excel-file.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/08/importing-pages-into-moss-2007-from-an-excel-file.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

In my [previous post](/blog/jjameson/2009/10/08/web-application-at-could-not-be-found-error-on-moss-2007-x64), I briefly introduced the concept of a utility to import pages  into Microsoft Office SharePoint Server (MOSS) 2007 from an Excel input file. This  can be very useful for Development and Test environments (where you frequently rebuild  the sites during the development process), as well as when migrating legacy Web  sites to a Production environment -- or whenever a "quick and dirty" process is  deemed sufficient for the business needs.

I've done a number of MOSS 2007 migration projects over the last several years  and developed a number of tools to migrate content into SharePoint. About a month  ago, I utilized some existing code I had previously written to import pages from  a simple Excel workbook -- rather than some other data source, such as a legacy  database. The Excel workbook simply contains one or more worksheets, where each  worksheet represents a particular group of pages with similar columns/fields. Each  worksheet must have the following columns:

- **WebUrl**
- **PageUrlName**
- **PageLayout**
- **Title**

Additional columns (such as **Page Content**) can also be specified  -- including custom columns for content types that are not included out-of-the-box  with MOSS 2007.

Note that I am not advocating installing the full Microsoft Office suite -- or  even just Excel -- on your SharePoint servers just for the sake of importing a few  hundred (or perhaps a few thousand) pages into your site. Rather, you would simply  need to install the following on one of the servers in your farm:

<cite>2007 Office System Driver: Data Connectivity Components</cite>
[http://www.microsoft.com/downloads/details.aspx?familyid=7554F536-8C28-4598-9B72-EF94E038C891&displaylang=en](http://www.microsoft.com/downloads/details.aspx?familyid=7554F536-8C28-4598-9B72-EF94E038C891&displaylang=en)

Preferably, you would install this only on a backend "job/index" server that  isn't servicing any user requests directly and then run the ImportPages.exe utility  on that server -- or at least, that's what I thought originally.

The problem is that in order to utilize the currently available OLEDB providers  (which are 32-bit), you have to force the process to run as 32-bit (by setting the  platform target to **x86**). However, as noted in my previous post,  you can't utilize the SharePoint API from a 32-bit process running on an x64 environment.  Quite the paradox indeed.

The solution that I came up with is to enhance the ImportPages.exe utility to  support an Excel input file as well as a simple XML file that contains a serialized  DataSet.

In other words, importing pages from Excel in an x64 environment requires a two-step  process:

1. Convert the Excel input file into a simple XML file that contains a serialized
   DataSet. Note that this can be done on any environment (i.e. it does not necessarily
   have to be done on one of the SharePoint servers). In other words, you could
   have someone create the Excel input file on his or her laptop and then convert
   it to a DataSet XML file.
2. Import the pages from the DataSet XML file. Note that this step must be
   performed on one of the SharePoint servers in the farm.

To convert an Excel file to a DataSet XML file:

```
ConvertToDataSet.exe Sample.xslx
```

To import pages from the generated DataSet XML file:

```
ImportPages.exe http://fabrikam Sample.xml
```

Here is the code for the ConvertToDataSet.exe utility:

```
using System;
using System.Data;
using System.IO;

using Fabrikam.Demo.CoreServices;

namespace Fabrikam.Demo.Tools.ConvertToDataSet
{
    /// <summary>
    /// Main class for the ConvertToDataSet.exe utility.
    /// </summary>
    class Program
    {
        private Program() { } // all members are static

        private static void Main(
            string[] args)
        {
            if (args.Length != 1)
            {
                Console.Error.WriteLine(
                    "Usage: ConvertToDataSet"
                    + " {filename}");

                Environment.Exit(1);
            }

            string inputFileName = args[0];

            try
            {
                string fileExtension = Path.GetExtension(inputFileName);

                if (string.Compare(
                    fileExtension,
                    ".xml",
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    throw new ArgumentException(
                        "Cannot convert an XML file to a DataSet.");
                }

                string outputFileName = Path.ChangeExtension(inputFileName, "xml");

                DataSet data = DataSetHelper.LoadFromFile(inputFileName);

                data.WriteXml(outputFileName);

                Console.WriteLine(
                    "DataSet serialized to file: {0}",
                    outputFileName);

                Environment.Exit(0);
            }
            catch (Exception ex)
            {
                Console.Error.WriteLine(ex.Message);

#if DEBUG
                Console.Error.WriteLine("(stack trace):");
                Console.Error.WriteLine(ex.StackTrace);
#endif

                Environment.Exit(1);
            }
        }
    }
}
```

Note that the bulk of the code has been refactored into my `DataSetHelper`  class:

```
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.OleDb;
using System.Globalization;
using System.IO;

namespace Fabrikam.Demo.CoreServices
{
    /// <summary>
    /// Exposes static methods for commonly used helper functions for
    /// the <see cref="System.Data.DataSet" /> class.
    /// This class cannot be inherited.
    /// </summary>
    /// <remarks>
    /// All methods of the <c>DataSetHelper</c> class are static and can
    /// therefore be called without creating an instance of the class.
    /// </remarks>
    public sealed class DataSetHelper
    {
        private DataSetHelper() { } // all members are static

        /// <summary>
        /// Loads a DataSet from the specified file.
        /// </summary>
        /// <remarks>The input file may be an Excel file (.xls or .xlsx) or a
        /// simple CSV file (i.e. comma-separated values).</remarks>
        /// <param name="fileName">The filename (including the path) from which
        /// to load the data.</param>
        /// <returns>A <see cref="System.Data.DataSet" /> containing the data
        /// read from the file.</returns>
        public static DataSet LoadFromFile(
            string fileName)
        {
            if (fileName == null)
            {
                throw new ArgumentNullException("fileName");
            }
            else if (string.IsNullOrEmpty(fileName) == true)
            {
                throw new ArgumentException(
                    "The file name must be specified.",
                    "fileName");
            }

            FileInfo inputFileInfo = new FileInfo(fileName);

            string connectionString = GetOleDbConnectionString(inputFileInfo);

            OleDbConnection connection = new OleDbConnection(connectionString);
            connection.Open();

            string[] tableNames = GetTableNames(inputFileInfo, connection);

            DataSet data = new DataSet();
            data.Locale = CultureInfo.InvariantCulture;

            foreach (string tableName in tableNames)
            {
                DataTable table = data.Tables.Add(tableName);

                string commandText = string.Format(
                    CultureInfo.InvariantCulture,
                    "SELECT * FROM [{0}]",
                    tableName);

                OleDbCommand selectCommand = new OleDbCommand(
                    commandText,
                    connection);

                OleDbDataAdapter adapter = new OleDbDataAdapter(selectCommand);

                adapter.Fill(table);
            }

            return data;
        }

        private static string GetOleDbConnectionString(
            FileInfo inputFileInfo)
        {
            string connectionString = null;

            if (string.Compare(
                inputFileInfo.Extension,
                ".xls",
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                connectionString =
                    @"Provider=Microsoft.Jet.OLEDB.4.0;"
                    + "Data Source=" + inputFileInfo.FullName + ";"
                    + "Extended Properties=\"Excel 8.0;HDR=YES;\"";
            }
            else if (string.Compare(
                inputFileInfo.Extension,
                ".xlsx",
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                connectionString =
                    @"Provider=Microsoft.ACE.OLEDB.12.0;"
                    + "Data Source=" + inputFileInfo.FullName + ";"
                    + "Extended Properties=\"Excel 12.0;HDR=Yes;ReadOnly=true;"
                    + "IMEX=1;\"";
            }
            else
            {
                string folder = inputFileInfo.DirectoryName;

                connectionString =
                    @"Provider=Microsoft.Jet.OLEDB.4.0;"
                    + "Data Source=" + folder + ";"
                    + "Extended Properties=\"text;HDR=YES;FMT=Delimited;\"";

            }

            return connectionString;
        }

        private static string[] GetTableNames(
            FileInfo inputFileInfo,
            OleDbConnection connection)
        {
            bool isExcelFile = false;

            if (string.Compare(
                inputFileInfo.Extension,
                ".xls",
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                isExcelFile = true;
            }
            else if (string.Compare(
                inputFileInfo.Extension,
                ".xlsx",
                StringComparison.OrdinalIgnoreCase) == 0)
            {
                isExcelFile = true;
            }

            if (isExcelFile == true)
            {
                // Get all of the Table names from the Excel workbook 
                DataTable worksheetTables = connection.GetOleDbSchemaTable(
                    OleDbSchemaGuid.Tables,
                    null);

                List<string> tableNames = new List<string>();

                for (int i = 0; i < worksheetTables.Rows.Count; i++)
                {
                    string tableName =
                        (string)worksheetTables.Rows[i]["TABLE_NAME"];

                    if (string.Compare(
                        tableName,
                        "_xlnm#_FilterDatabase",
                        StringComparison.OrdinalIgnoreCase) == 0)
                    {
                        continue;
                    }

                    tableNames.Add(tableName);
                }

                return tableNames.ToArray();
            }
            else
            {
                string[] tableNames = new string[1];
                tableNames[0] = inputFileInfo.Name;
                return tableNames;
            }
        }
    }
}
```

Similarly, the bulk of the code for the ImportPages.exe utility has been refactored  into the `PageImporter` and `SharePointPublishingHelper` classes.

Here's the code for the `PageImporter` class:

```
using System;
using System.Data;
using System.Diagnostics;
using System.Globalization;
using System.IO;

using Microsoft.SharePoint;
using Microsoft.SharePoint.Publishing;

using Fabrikam.Demo.CoreServices;
using Fabrikam.Demo.CoreServices.Logging;
using Fabrikam.Demo.CoreServices.SharePoint;

namespace Fabrikam.Demo.Tools.SharePoint.ImportPages
{
    /// <summary>
    /// Exposes static methods for importing pages into a SharePoint site.
    /// This class cannot be inherited.
    /// </summary>
    /// <remarks>
    /// All methods of the <c>PageImporter</c> class are static and can
    /// therefore be called without creating an instance of the class.
    /// </remarks>    
    class PageImporter
    {
        private PageImporter() { } // all members are static 

        /// <summary>
        /// Imports pages into the specified SharePoint site using data from
        /// the specified input file.
        /// </summary>
        /// <param name="siteUrl">The URL of the SharePoint site.</param>
        /// <param name="fileName">The filename (including the path) from which
        /// to load the data.</param>
        public static void Import(
            string siteUrl,
            string fileName)
        {
            DataSet data = DataSetHelper.LoadFromFile(fileName);

            using (SPSite site = new SPSite(siteUrl))
            {
                Import(site, data);
            }
        }

        private static void Import(
            SPSite site,
            DataSet data)
        {
            foreach (DataTable table in data.Tables)
            {
                Import(site, table);
            }
        }

        private static void Import(
            SPSite site,
            DataTable table)
        {
            SPWeb web = null;

            foreach (DataRow row in table.Rows)
            {
                string webUrl = (string)row["WebUrl"];
                string pageUrlName = (string)row["PageUrlName"];

                try
                {
                    EnsureReferenceToExpectedWeb(site, ref web, webUrl);

                    Import(web, row);
                }
                catch (ArgumentNullException ex)
                {
                    Logger.LogWarning(
                        CultureInfo.InvariantCulture,
                        "Error importing page ({0}) into Web ({1}) - {2}.",
                        pageUrlName,
                        webUrl,
                        ex.Message);
                }
                catch (FileNotFoundException ex)
                {
                    Logger.LogWarning(
                        CultureInfo.InvariantCulture,
                        "Error importing page ({0}) into Web ({1}) - {2}.",
                        pageUrlName,
                        webUrl,
                        ex.Message);
                }
            }

            if (web != null)
            {
                web.Dispose();
            }
        }

        private static void EnsureReferenceToExpectedWeb(
            SPSite site,
            ref SPWeb web,
            string expectedWebUrl)
        {
            Debug.Assert(string.IsNullOrEmpty(expectedWebUrl) == false);

            if (web == null)
            {
                web = site.OpenWeb(expectedWebUrl);
            }
            else if (string.Compare(
                web.ServerRelativeUrl,
                expectedWebUrl,
                StringComparison.OrdinalIgnoreCase) != 0)
            {
                web.Dispose();

                web = site.OpenWeb(expectedWebUrl);
            }
        }

        private static void Import(
            SPWeb web,
            DataRow row)
        {
            string pageUrlName = (string)row["PageUrlName"];

            string pageLayoutUrl =
                "_catalogs/masterpage/" + (string)row["PageLayout"];

            string pageTitle = null;
            if (row.IsNull("Title") == false)
            {
                pageTitle = (string)row["Title"];

                if (pageTitle.Length == 255)
                {
                    Logger.LogWarning(
                        CultureInfo.InvariantCulture,
                        "The title specified for the page ({0}) may have been"
                            + " truncated, since it is exactly 255 characters.",
                        pageUrlName);
                }
            }

            PublishingPage page = SharePointPublishingHelper.EnsurePage(
                web,
                pageUrlName,
                pageTitle,
                pageLayoutUrl);

            bool skipPageConfiguration =
                CheckIfPageImportShouldBeSkipped(
                    page);

            if (skipPageConfiguration == true)
            {
                return;
            }

            SharePointPublishingHelper.EnsurePageIsCheckedOutToMe(
                page);

            page.Title = pageTitle;

            foreach (DataColumn column in row.Table.Columns)
            {
                string columnName = column.ColumnName;

                if (string.Compare(
                    columnName,
                    "WebUrl",
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    continue;
                }
                else if (string.Compare(
                    columnName,
                    "PageUrlName",
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    continue;
                }
                else if (string.Compare(
                    columnName,
                    "PageLayout",
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    continue;
                }
                else if (string.Compare(
                    columnName,
                    "Title",
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    continue;
                }

                if (row.IsNull(columnName) == false)
                {
                    page.ListItem[columnName] = row[columnName];
                }
            }

            page.Update();

            SharePointPublishingHelper.PublishPage(
                page,
                "Published by Fabrikam.Demo.Tools.ImportPages.PageImporter.");
        }

        private static bool CheckIfPageImportShouldBeSkipped(
            PublishingPage page)
        {
            if (SharePointPublishingHelper.IsPageApproved(page))
            {
                Logger.LogInfo(
                    CultureInfo.InvariantCulture,
                    "Skipping import of page ({0}/{1}) because the page"
                        + " is approved.",
                    page.ListItem.Web.Url,
                    page.Url);

                return true;
            }

            bool isPageCheckedOut =
                SharePointPublishingHelper.IsPageCheckedOut(
                    page);

            bool isPageCheckedOutToMe =
                SharePointPublishingHelper.IsPageCheckedOutToMe(
                    page);

            if (isPageCheckedOut == true
                && isPageCheckedOutToMe == false)
            {
                Logger.LogWarning(
                    CultureInfo.InvariantCulture,
                    "Skipping import of page"
                        + " ({0}/{1}) because the page is already checked"
                        + " out to somebody else.",
                    page.ListItem.Web.Url,
                    page.Url);

                return true;
            }

            return false;
        }
    }
}
```

I'll cover the details of the `SharePointPublishingHelper` class in  a [separate post](/blog/jjameson/2009/10/09/introducing-the-sharepointpublishinghelper-class).

