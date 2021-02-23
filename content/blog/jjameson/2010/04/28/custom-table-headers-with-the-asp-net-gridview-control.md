---
title: "Custom Table Headers With the ASP.NET GridView Control"
date: 2010-04-28T07:07:00-06:00
excerpt: "In my previous post , I showed an example KPI dashboard for a Web application with a table similar to the following: 
 
 Key Performance Indicators (Detail) 
 
 Site 2009 Q3 2009 Q4 2010 Q1 Thresholds 
 
 
 
 
 
 
 
 Duncan 
 93% 
 95% ..."
aliases: ["/blog/jjameson/archive/2010/04/27/custom-table-headers-with-the-asp-net-gridview-control.aspx", "/blog/jjameson/archive/2010/04/28/custom-table-headers-with-the-asp-net-gridview-control.aspx"]
draft: true
categories: ["Development"]
tags: ["Web Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/28/custom-table-headers-with-the-asp-net-gridview-control.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/28/custom-table-headers-with-the-asp-net-gridview-control.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

In my [previous post](/blog/jjameson/2010/04/22/still-crazy-about-typed-datasets-after-all-these-years), I showed an example KPI dashboard for a Web application with  a table similar to the following:

{{< table class="small" caption="Key Performance Indicators (Detail)" >}}

| Site | 2009 Q3 | 2009 Q4 | 2010 Q1 | Thresholds |
| --- | --- | --- | --- | --- |
| ![Exceeds](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-0-16x16.gif) | ![Meets](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-1-16x16.gif) | ![Does Not Meet](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-2-16x16.gif) |
| --- | --- | --- |
| Duncan | 93% | 95% | 92% | &gt;= 90% | 86% - 90% | &lt;= 85% |
| Dallas | 94% ![(Different KPI Thresholds)](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Kpi_ShowProblems-16x16.gif "The KPI thresholds for this period were different from the current period. (Exceeds: >= 90%, Meets: 86% - 90%, Does Not Meet: <= 85%)") | 91% | 90% | &gt;= 92% | 88% - 92% | &lt;= 88% |
| Albuquerque | 91% | 87% | 85% | &gt;= 90% | 86% - 90% | &lt;= 85% |
| Denver | 94% | 91% | 92% | &gt;= 90% | 86% - 90% | &lt;= 85% |

{{< /table >}}

I also hinted that it took a little more work than I expected to render the custom  header for the table (i.e. the **Thresholds** label that spans the  last three columns). Note that I originally mocked up the KPI feature using a static  HTML prototype, so I knew what I wanted the underlying HTML markup to look like.  However, the trick was to generate the desired HTML using an ASP.NET control.

I had originally planned on using the Telerik RadGrid control to render the KPI  detail table (since we are using it in a variety of other features on the site).  However, as I mentioned in my earlier post, I eventually gave up trying to customize  the table header on the RadGrid and instead replaced it with the out-of-the-box  GridView control in ASP.NET. [While I did find [one approach for rendering a RadGrid control with two header rows](http://www.telerik.com/community/forums/aspnet-ajax/grid/radgrid-custom-header-with-tow-rows.aspx), I also discovered  issues with that approach when using auto-generated columns.]

Note that the number of columns displayed in the KPI detail table may vary. In  the example shown above, there are three "period" columns, corresponding to "2009  Q3", "2009 Q4", and "2010 Q1". However, what if we only had data available for the  "2010 Q1" period (or we wanted to show data from more than three periods)?

To make the presentation layer as flexible as possible, I chose to auto-generate  the columns and rely on a couple of assumptions when rendering the table:

- The first column will always contain the site names.
- The last three columns will always contain the KPI thresholds from the most
  recent period. (Note that the business may decide to modify the KPI thresholds
  over time, but we'll always display the most recent KPI thresholds in the detail
  table.)

Consequently, we'll assume the underlying data source -- in this case, a simple  DataTable -- that is bound to the grid control specifies the exact columns that  we want to show in the table, and therefore we can defer to the grid control to  auto-generate the necessary columns. In the example above, the DataTable contains  the following columns:

- Site
- 2009 Q3
- 2009 Q4
- 2010 Q1
- Threshold - Exceeds
- Threshold - Meets
- Threshold - Does Not Meet

As I usually do when building a feature, let's start with a basic implementation  and then iterate the code until we've completed the scenario.

> **Tip**
>
> The KPI dashboard is actually displayed in a customer portal based on Microsoft
> Office SharePoint Server (MOSS) 2007. However, my recommendation when creating
> a feature like this for a SharePoint application, is to first get it working
> in a simple ASP.NET application (using sample data), and, once it is working
> to your satisfaction, get it to render on a SharePoint site (using real
> data). If you try to do all of the development exclusively through your
> SharePoint Web application, you'll spend far more time iterating the development
> (for example, deploying updated files, GAC'ing assemblies, or recycling
> your app pool).

So let's start with a simple ASP.NET user control (KpiScorecard.ascx) that encapsulates  the presentation layer for the KPI scorecard:

```
<%@ Control Language="C#" AutoEventWireup="true" CodeBehind="KpiScorecard.ascx.cs"
    Inherits="Fabrikam.Demo.Web.UI.Tables.KpiScorecard" %>
<div class="kpiScorecard">
    <div class="webPartToolbar">
        <asp:LinkButton runat="server" ID="PostBackButton" Text="Post Back" />
    </div>
    <asp:GridView runat="server" ID="ScorecardDetailView" OnRowCreated="ScorecardDetailView_RowCreated" />
</div>
```

In the corresponding code-behind file, we'll create some sample data (which will  eventually be retrieved from a services layer) and bind it to the GridView control:

```
using System;
using System.Data;
using System.Globalization;

namespace Fabrikam.Demo.Web.UI.Tables
{
    public partial class KpiScorecard : System.Web.UI.UserControl
    {
        protected void Page_Load(
            object sender,
            EventArgs e)
        {
            if (this.Page.IsPostBack == true)
            {
                // Render the KPI scorecard from view state
                return;
            }

            UpdateScorecardDetailView();
        }

        private static DataTable GetScorecardDetailTable()
        {
            DataTable detailTable = new DataTable();
            detailTable.Locale = CultureInfo.CurrentCulture;

            detailTable.Columns.Add("Site");
            detailTable.Columns.Add("2009 Q3");
            detailTable.Columns.Add("2009 Q4");
            detailTable.Columns.Add("2010 Q1");
            detailTable.Columns.Add("Threshold - Exceeds");
            detailTable.Columns.Add("Threshold - Meets");
            detailTable.Columns.Add("Threshold - Does Not Meet");

            detailTable.Rows.Add(
                new object[]
                {
                    "Duncan",
                    "93%",
                    "95%",
                    "92%",
                    ">= 90%",
                    "86% - 90%",
                    "<= 85%"
                });

            detailTable.Rows.Add(
                new object[]
                {
                    "Dallas ",
                    "94%",
                    "91%",
                    "90%",
                    ">= 92%",
                    "88% - 92%",
                    "<= 88%"
                });

            detailTable.Rows.Add(
                new object[]
                {
                    "Albuquerque",
                    "91%",
                    "87%",
                    "85%",
                    ">= 90%",
                    "86% - 90%",
                    "<= 85%"
                });

            detailTable.Rows.Add(
                new object[]
                {
                    "Denver",
                    "94%",
                    "91%",
                    "92%",
                    ">= 90%",
                    "86% - 90%",
                    "<= 85%"
                });

            return detailTable;
        }

        private void UpdateScorecardDetailView()
        {
            using (DataTable detailTable = GetScorecardDetailTable())
            {
                ScorecardDetailView.DataSource = detailTable;
                ScorecardDetailView.DataBind();
            }
        }
    }
}
```

A couple of important points about the implementation:

- Assume the actual work required to get the scorecard detail table is substantial
  (e.g. the real data will be retrieved from an external Web service or database).
  Consequently, in the `Page_Load` event handler, we check if it is
  the not the initial page request (i.e. `this.Page.IsPostBack  == true`), in which case we rely on
  the GridView to restore its content from view state.
- A simple LinkButton allows us to test the scenario where some control on
  the page causes a post back to the server.

It turns out that rendering the GridView from view state is actually what makes  adding a custom header row interesting enough for a blog post. I'll explain why  in a moment.

Running the Web application at this point renders a simple table similar to the  following:

{{< table class="small" >}}

| Site | 2009 Q3 | 2009 Q4 | 2010 Q1 | Threshold - Exceeds | Threshold - Meets | Threshold - Does Not Meet |
| --- | --- | --- | --- | --- | --- | --- |
| Duncan | 93% | 95% | 92% | &gt;= 90% | 86% - 90% | &lt;= 85% |
| Dallas  | 94% | 91% | 90% | &gt;= 92% | 88% - 92% | &lt;= 88% |
| Albuquerque | 91% | 87% | 85% | &gt;= 90% | 86% - 90% | &lt;= 85% |
| Denver | 94% | 91% | 92% | &gt;= 90% | 86% - 90% | &lt;= 85% |

{{< /table >}}

Let's start customizing the header by replacing the lengthy column headings for  the KPI thresholds with corresponding icons. This is easily achieved using a little  bit of code in the **[GridView.RowCreated](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.gridview.rowcreated.aspx)** event:

```
protected void ScorecardDetailView_RowCreated(
            object sender,
            GridViewRowEventArgs e)
        {
            if (e == null)
            {
                throw new ArgumentNullException("e");
            }

            if (e.Row.RowType == DataControlRowType.Header)
            {
                e.Row.Cells[e.Row.Cells.Count - 1].Text =
                    "<img src='/_layouts/images/kpidefault-2.gif'"
                        + " alt='Does Not Meet' />";

                e.Row.Cells[e.Row.Cells.Count - 2].Text =
                    "<img src='/_layouts/images/kpidefault-1.gif'"
                        + " alt='Meets' />";

                e.Row.Cells[e.Row.Cells.Count - 3].Text =
                    "<img src='/_layouts/images/kpidefault-0.gif'"
                        + " alt='Exceeds' />";
            }
        }
```

> **Note**
>
> As I mentioned before, the KPI dashboard is actually displayed in a customer
> portal based on MOSS 2007. Consequently, I chose to reuse the KPI images
> that come out-of-the-box with MOSS 2007 (in case you were wondering why
> the image path refers to **\_layouts**).

Running the Web application at this point shows the images in place of the lengthy  column headings, similar to the following:

{{< table class="small" >}}

| Site | 2009 Q3 | 2009 Q4 | 2010 Q1 | ![Exceeds](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-0-16x16.gif) | ![Meets](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-1-16x16.gif) | ![Does Not Meet](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-2-16x16.gif) |
| --- | --- | --- | --- | --- | --- | --- |
| Duncan | 93% | 95% | 92% | &gt;= 90% | 86% - 90% | &lt;= 85% |
| Dallas  | 94% | 91% | 90% | &gt;= 92% | 88% - 92% | &lt;= 88% |
| Albuquerque | 91% | 87% | 85% | &gt;= 90% | 86% - 90% | &lt;= 85% |
| Denver | 94% | 91% | 92% | &gt;= 90% | 86% - 90% | &lt;= 85% |

{{< /table >}}

Now let's add a method to insert another row into the table rendered by the GridView  control:

```
private static void AddThresholdsHeaderRow(
            GridView scorecardDetailView)
        {
            Debug.Assert(scorecardDetailView != null);

            if (scorecardDetailView.Controls.Count < 1)
            {
                // No data to display in the grid (i.e. the user has not
                // selected a KPI in the summary table and therefore the
                // detail table has not yet been bound to any data)
                return;
            }

            Debug.Assert(scorecardDetailView.Controls.Count == 1);
            Table table = (Table)scorecardDetailView.Controls[0];
            TableRow headerRow = table.Rows[0];

            using (GridViewRow additionalHeaderRow = new GridViewRow(
                -1,
                -1,
                DataControlRowType.Header,
                DataControlRowState.Normal))
            {

                int numberOfHeaderCellsToMove = headerRow.Cells.Count - 3;

                for (int i = 0; i < numberOfHeaderCellsToMove; i++)
                {
                    TableCell headerCell = headerRow.Cells[0];
                    headerRow.Cells.RemoveAt(0);
                    additionalHeaderRow.Cells.Add(headerCell);
                    headerCell.RowSpan = 2;
                }

                using (TableHeaderCell newHeaderCell = new TableHeaderCell())
                {
                    newHeaderCell.ColumnSpan = 3;
                    newHeaderCell.Text = "Thresholds";
                    additionalHeaderRow.Cells.Add(newHeaderCell);
                }

                table.Controls.AddAt(
                    0,
                    additionalHeaderRow);
            }
        }
```

Of course, we obviously need to call this method, so let's modify the **UpdateScorecardDetailView** method to add the thresholds header row after  binding the GridView control:

```
private void UpdateScorecardDetailView()
        {
            using (DataTable detailTable = GetScorecardDetailTable())
            {
                ScorecardDetailView.DataSource = detailTable;
                ScorecardDetailView.DataBind();
            }

            AddThresholdsHeaderRow(ScorecardDetailView);
        }
```

Running the Web application at this point shows the **Thresholds**  header above the corresponding columns, similar to the following:

{{< table class="small" >}}

| Site | 2009 Q3 | 2009 Q4 | 2010 Q1 | Thresholds |
| --- | --- | --- | --- | --- |
| ![Exceeds](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-0-16x16.gif) | ![Meets](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-1-16x16.gif) | ![Does Not Meet](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-2-16x16.gif) |
| --- | --- | --- |
| Duncan | 93% | 95% | 92% | &gt;= 90% | 86% - 90% | &lt;= 85% |
| Dallas  | 94% | 91% | 90% | &gt;= 92% | 88% - 92% | &lt;= 88% |
| Albuquerque | 91% | 87% | 85% | &gt;= 90% | 86% - 90% | &lt;= 85% |
| Denver | 94% | 91% | 92% | &gt;= 90% | 86% - 90% | &lt;= 85% |

{{< /table >}}

Looking at the HTML source, we can see the extra table row has been inserted,  and the `rowspan` and `colspan` attributes are being rendered  as expected.

```
<table style="border-collapse: collapse"
        id="KpiScorecard1_ScorecardDetailView" border="1" rules="all"
        cellspacing="0">
        <tbody>
            <tr>
                <th rowspan="2" scope="col">
                    Site
                </th>
                <th rowspan="2" scope="col">
                    2009 Q3
                </th>
                <th rowspan="2" scope="col">
                    2009 Q4
                </th>
                <th rowspan="2" scope="col">
                    2010 Q1
                </th>
                <th colspan="3">
                    Thresholds
                </th>
            </tr>
            <tr>
                <th scope="col">
                    <img alt="Exceeds" src="/_layouts/images/kpidefault-0.gif">
                </th>
                <th scope="col">
                    <img alt="Meets" src="/_layouts/images/kpidefault-1.gif">
                </th>
                <th scope="col">
                    <img alt="Does Not Meet" src="/_layouts/images/kpidefault-2.gif">
                </th>
            </tr>
            ...
        </tbody>
    </table>
```

At this point, it seems like we are done, right? At least in regards to adding  a row to the table header (we obviously still need to specify CSS classes on various  table elements in order to achieve the desired styling -- however, that wasn't the  intent of this post).

The problem is that there a couple of bugs in the implementation.

What happens when a post back occurs on the page? Recall that I originally added  a LinkButton to test this very scenario. If you click the button (to cause a post  back), the GridView renders itself from view state, similar to the following:

{{< table class="small" >}}

| Site | 2009 Q3 | 2009 Q4 | 2010 Q1 | ![Exceeds](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-0-16x16.gif) | ![Meets](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-1-16x16.gif) | ![Does Not Meet](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-2-16x16.gif) |
| --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |  |
| Duncan | 93% | 95% | 92% | &gt;= 90% | 86% - 90% | &lt;= 85% |
| Dallas  | 94% | 91% | 90% | &gt;= 92% | 88% - 92% | &lt;= 88% |
| Albuquerque | 91% | 87% | 85% | &gt;= 90% | 86% - 90% | &lt;= 85% |

{{< /table >}}

Notice that we no longer see the **Thresholds** header in the table.  This is because the **UpdateScorecardDetailView** method is not called  on post back, and the custom header row that we inserted before is not serialized  in view state. [The table also appears to be "corrupted" -- meaning some extraneous  cells appear on the right side of the table.]

My initial attempt at fixing this bug was to call the **AddThresholdsHeaderRow** method in the PreRenderComplete phase of the page instead of from the **UpdateScorecardDetailView** method (to force the header row to be  added regardless of whether we are binding the GridView to a data source or rendering  it from view state):

```
protected void Page_Load(
            object sender,
            EventArgs e)
        {
            this.Page.PreRenderComplete += new EventHandler(Page_PreRenderComplete);

            if (this.Page.IsPostBack == true)
            {
                // Render the KPI scorecard from view state
                return;
            }

            UpdateScorecardDetailView();
        }

        void Page_PreRenderComplete(
            object sender,
            EventArgs e)
        {
            AddThresholdsHeaderRow(ScorecardDetailView);
        }
```

Upon first inspection, this appeared to work because the **Thresholds**  header is added on the initial page request, as well as upon post back. However,  there's still a problem...

Take another look at the table above. What happened to the row containing the  details for the **Denver** site?

It turns out that adding a custom header row to a GridView (using the approach  I've discussed here) corrupts the view state for the control. Consequently, if you  rely on the GridView to render properly from view state, then you have a nasty bug.  [On the other hand, if you don't need or want to render the GridView from view state,  then at this point, you can consider it "good enough" and move on to your next development  task.]

To resolve this bug, we need to find a way to avoid corrupting the view state  of the GridView control, while still adding a custom header row.

It turns out this is really easy. Instead of adding the header row during the  PreRenderComplete phase of the page, let's instead call the **AddThresholdsHeaderRow** method in the SaveStateComplete phase:

```
protected void Page_Load(
                    object sender,
                    EventArgs e)
        {
            this.Page.SaveStateComplete += new EventHandler(Page_SaveStateComplete);

            if (this.Page.IsPostBack == true)
            {
                // Render the KPI scorecard from view state
                return;
            }

            UpdateScorecardDetailView();
        }

        void Page_SaveStateComplete(
            object sender,
            EventArgs e)
        {
            AddThresholdsHeaderRow(ScorecardDetailView);
        }
```

Here's the short summary from MSDN for the [Page.SaveStateComplete](http://msdn.microsoft.com/en-us/library/system.web.ui.page.savestatecomplete.aspx) event:

{{< blockquote "font-italic" >}}

Occurs after the page has completed saving all view state and control state information for the page and controls on the page.

{{< /blockquote >}}

With this change, the KPI detail table renders as expected (with the custom **Thresholds** header row and all of the expected data) even when the  GridView is rendered from view state.

> **Update (2011-04-21)**
>
> I've attached a sample Visual Studio solution to make it easier to see this
> concept in action.

