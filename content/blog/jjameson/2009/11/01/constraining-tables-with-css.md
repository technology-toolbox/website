---
title: "Constraining Tables with CSS"
date: 2009-11-01T05:15:00-07:00
lastmod: 2009-11-01T06:15:00-07:00
excerpt: "Have you ever wanted to display data in a table but limit the size of the rows and columns within the table? 
 For example, consider the classic master/detail view that we often find in software applications, in which items are shown in a summary table..."
aliases: ["/blog/jjameson/archive/2009/10/31/constraining-tables-with-css.aspx", "/blog/jjameson/archive/2009/11/01/constraining-tables-with-css.aspx"]
draft: true
categories: ["Development"]
tags: ["Web Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/11/01/constraining-tables-with-css.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/11/01/constraining-tables-with-css.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Have you ever wanted to display data in a table but limit the size of the rows
and columns within the table?

For example, consider the classic master/detail view that we often find in
software applications, in which items are shown in a summary table and each row
provides a link to allow users to see more detail about the item.

However, unlike the typical master/detail scenario, you need to limit the amount
of real estate consumed on the page by the summary table. In addition, if text
within a column is too long to fit within the constrained area, you want to show
the full text when the mouse cursor hovers over the cell.

The following figure illustrates the desired end result:

{{< figure src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Constrained-Table-559x166.png" alt="Constrained table" height="166" width="559" title="Figure 1: Constrained table" >}}

Here is the sample ASP.NET page that I created this morning to demonstrate this:

```
<%@ Page Language="C#" AutoEventWireup="true"    CodeBehind="ConstrainedTable.aspx.cs"    Inherits="Fabrikam.Demo.Web.UI.ConstrainedTable" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"                                "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" >
<head runat="server">
    <title>Demo - Constrained Tabular Data</title>
    <link rel="Stylesheet" href="http://meyerweb.com/eric/tools/css/reset/reset.css" />
    <style>
        /* Basic formatting
        ----------------------------------------------------------------------*/
        body {
            color:#000000;
            font-family: Verdana, Arial, Helvetica;
            font-size: small;
            margin: 10px;
        }
        h1, h2, h3, h4, h5, h6, strong {
            font-weight: bold;
        }
        h1 {
            font-size: 1.5em;
            margin: .67em 0;
        }
        table caption {
            font-weight: bold;
            margin: 10px 0;
        }
        table.displayTable {
            border-left: 1px solid #36295E;
            border-right: 1px solid #36295E;
            margin: 1em 0;
        }
        table.displayTable th {
            text-align: left;
        }
        table.displayTable th, table.displayTable td {
            border-bottom: 1px solid #36295E;
            padding: 5px 10px;
            vertical-align: top;
        }
        table.displayTable thead th {
            background: #36295E;
            color: #FFF;
            padding: 8px 10px;
            border-bottom-width: 0;
        }
        table.displayTable tr.altRow {
            background: #F4F4F4;
        }
        /* =constrainedTable
        ----------------------------------------------------------------------*/
        table.constrainedTable {
            table-layout: fixed;
            width: 540px;
        }
        table.constrainedTable th.nameColumn {
            width: 140px;
        }
        table.constrainedTable td {
            overflow: hidden;
            -o-text-overflow: ellipsis; /* Opera */
            text-overflow: ellipsis;
            white-space: nowrap;
        }
    </style>
</head>
<body>
    <form id="form1" runat="server">
        <h1>Demo - Constrained Tabular Data</h1>
        <asp:GridView ID="ConstrainedGrid" runat="server"
            AutoGenerateColumns="false"
            Caption="Constrained Table"
            CssClass="displayTable constrainedTable"
            OnRowDataBound="ConstrainedGrid_RowDataBound"
            EnableViewState="false"
            UseAccessibleHeader="true">
            <AlternatingRowStyle CssClass="altRow" />
            <RowStyle CssClass="row" />
            <Columns>
                <asp:HyperLinkField
                    DataTextField="Site"
                    DataNavigateUrlFields="URL"
                    HeaderText="Site"
                    HeaderStyle-CssClass="nameColumn" />
                <asp:BoundField
                    DataField="Notes"
                    HeaderText="Notes"
                    HeaderStyle-CssClass="notesColumn" />
            </Columns>
        </asp:GridView>
    </form>
</body>
</html>
```

The most interesting parts of the ASP.NET page are the CSS rules for the
`constrainedTable` class:

```
table.constrainedTable {
    table-layout: fixed;
    width: 540px;
}
table.constrainedTable th.nameColumn {
    width: 140px;
}
table.constrainedTable td {
    overflow: hidden;
    -o-text-overflow: ellipsis; /* Opera */
    text-overflow: ellipsis;
    white-space: nowrap;
}
```

Changing the `table-layout` to `fixed` constrains the table to the specified
width. Since I don't specify a width for the `notesColumn` it consumes the
remaining width of the table.

Next, I specify that all cells in the constrained table should truncate the text
within each cell if it is too wide to fit within the width of the column. This
is achieved using the combination of `overflow: hidden` and `white-space: nowrap`. Finally, I use the ` text-overflow` CSS property to show ellipsis when
text within a cell is clipped (as well as a slight variation for the Opera
browser).

> **Note**
>
> Support for the `text-overflow` CSS property is somewhat limited. In
> particular, you will find that a clipped table cell renders without the
> ellipsis in Firefox (at least in version 3.5.3). Oddly enough, this appears to
> have been supported in Internet Explorer since version 6. In addition to
> Internet Explorer and Opera, this also appears as expected in Safari.

Also note that constraining table cells using CSS does not automatically display
the tooltip with the complete text. For that, you are on your own.

When using an ASP.NET GridView control, I recommend using the
[RowDataBound](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.gridview.rowdatabound.aspx)
event in order to set the
[ToolTip](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.webcontrol.tooltip.aspx)
property of the table cell. Just be sure to decode the contents of the cell to
avoid having it encoded twice:

```
        protected void ConstrainedGrid_RowDataBound(
            object sender,
            GridViewRowEventArgs e)
        {
            foreach (TableCell cell in e.Row.Cells)
            {
                // Note: We need to decode the cell text in order to avoid
                // having it encoded twice (e.g. "&amp;gt;")
                cell.ToolTip = HttpUtility.HtmlDecode(cell.Text);
            }
        }
```

This is somewhat of a "brute force" approach since it makes no attempt to
determine if the text will be truncated, but rather sets the ToolTip (in other
words, the title attribute on the HTML element) for every cell. However, the
simplicity of this approach -- and the resulting user experience -- far
outweighs any extraneous markup (at least in my opinion).

Be aware that this simple approach for setting the ToolTip property doesn't
support columns generated from a
[HyperLinkField](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.hyperlinkfield.aspx),
in which case `cell.Text` is empty (because the cell contains child controls,
not simple text). In other words, it's a good thing my demo doesn't specify very
long site names that don't fit within the 140 pixel width column constraint ;-)

Here is the complete code-behind for my sample ASP.NET page so you can run it
yourself:

```
using System;
using System.Data;
using System.Globalization;
using System.Web;
using System.Web.UI.WebControls;

namespace Fabrikam.Demo.Web.UI
{
    public partial class ConstrainedTable : System.Web.UI.Page
    {
        protected void Page_PreRender(
            object sender,
            EventArgs e)
        {
            BindSampleData(this.ConstrainedGrid);

            this.ConstrainedGrid.HeaderRow.TableSection =
                TableRowSection.TableHeader;
        }

        #region BindSampleData

        private static void BindSampleData(
            GridView grid)
        {
            DataTable sampleData = new DataTable();
            sampleData.Locale = CultureInfo.InvariantCulture;

            sampleData.Columns.Add("Site");
            sampleData.Columns.Add("URL");
            sampleData.Columns.Add("Notes");

            sampleData.Rows.Add(
                new object[]
                {
                    "Microsoft",
                    "http://www.microsoft.com",
                    "Lorem ipsum dolor sit amet, consectetur adipiscing elit."
                        + "Morbi at sem lorem, ac blandit leo. Phasellus at"
                        + "ligula vitae enim dignissim tincidunt ornare nisl."
                });

            sampleData.Rows.Add(
                new object[]
                {
                    "MSDN",
                    "http://msdn.microsoft.com",
                    "Sed posuere mattis egestas. Aliquam commodo dolor"
                        + " vulputate odio lacinia bibendum. Nullam bibendum,"
                        + " neque vitae ullamcorper elementum, ligula dolor"
                        + " mollis erat, ut ultricies mauris tortor ut eros."
                });

            sampleData.Rows.Add(
                new object[]
                {
                    "TechNet",
                    "http://technet.microsoft.com",
                    "Vestibulum a leo nisl, sit amet porta eros. Proin vitae"
                        + " semper nunc. In facilisis nunc sit amet lacus"
                        + " accumsan mattis. Nulla facilisi. Pellentesque nisl"
                        + " sapien, dignissim ultrices semper et, mollis"
                        + " interdum sem. "
                });

            grid.DataSource = sampleData;
            grid.DataBind();
        }

        #endregion

        protected void ConstrainedGrid_RowDataBound(
            object sender,
            GridViewRowEventArgs e)
        {
            foreach (TableCell cell in e.Row.Cells)
            {
                // Note: We need to decode the cell text in order to avoid
                // having it encoded twice (e.g. "&amp;gt;")
                cell.ToolTip = HttpUtility.HtmlDecode(cell.Text);
            }
        }
    }
}
```

> **Update (2011-04-21)**
>
> I've attached a sample Visual Studio solution to make it easier to see this
> concept in action.

