---
title: Leveraging the Power of Typed DataSets, IEnumerable<>, and LINQ
date: 2010-04-24T07:59:00-06:00
excerpt:
  In my previous post , I extolled my love of typed DataSets in .NET because
  they are not only quick to design and update, but also very easy to
  understand. Essentially, if you can read an entity-relationship model
  (&agrave; la ERwin or a Visio database...
aliases:
  [
    "/blog/jjameson/archive/2010/04/23/leveraging-the-power-of-typed-datasets-ienumerable-and-linq.aspx",
    "/blog/jjameson/archive/2010/04/24/leveraging-the-power-of-typed-datasets-ienumerable-and-linq.aspx",
  ]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/24/leveraging-the-power-of-typed-datasets-ienumerable-and-linq.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/24/leveraging-the-power-of-typed-datasets-ienumerable-and-linq.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In my
[previous post](/blog/jjameson/2010/04/22/still-crazy-about-typed-datasets-after-all-these-years),
I extolled my love of typed DataSets in .NET because they are not only quick to
design and update, but also very easy to understand. Essentially, if you can
read an entity-relationship model (&agrave; la ERwin or a Visio database
diagram) -- which I suspect nearly all developers can -- then you can discern
quite a bit of information from a typed DataSet.

Here is the example typed DataSet I introduced in the previous post.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Typed-DataSet-example-(ScorecardData)-600x330.png"
alt="Typed DataSet example (ScorecardData)" height="330" width="600"
title="Figure 1: Typed DataSet example (ScorecardData)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Typed-DataSet-example-%28ScorecardData%29-723x398.png)

Suppose that we need to display KPI scorecard information in a Web application.
How should we go about leveraging the typed DataSet?

Let's start with the presentation layer by creating an ASP.NET user control to
encapsulate the UI for KPI scorecards (KpiScorecard.ascx). On the left side of
the scorecard, we'll show a "summary view" and on the right side -- when a user
selects one of the KPIs in the summary view -- we'll show a "detatil view."
Pretty standard stuff, right?

We obviously want to keep the presentation layer as thin as possible (to make it
faster to develop the solution and also make it easier to test and maintain), so
let's use a service layer to perform the bulk of the work for retrieving and
manipulating an instance of the **ScorecardData** DataSet. Also note that,
ideally, we don't want the presentation layer to know anything about where the
data comes from or how it gets populated.

Consequently, we create a class called **ScorecardService** and add a method to
return a DataSet given a list of client sites (i.e. an array of site IDs --
where each site ID is a `string`):

```
    public static class ScorecardService
    {
        public static ScorecardData GetScorecardData(
            string[] sites)
        {
            ...
        }
    }
```

Note that the presentation layer simply needs to know a list of sites to
specify. Whether the scorecard data comes directly from a database, an external
Web service, or something else entirely doesn't matter from the presentation
perspective. We just need to get some scorecard data and display it.

As I mentioned before, suppose we want to show a summary view on the left side
of the page, perhaps with a list of KPI names along with rollup status values
across all sites for various time periods. Rather than aggregating the scorecard
data in the presentation layer, we can instead add another method to the
**ScorecardService** class:

```
        public static DataTable GetScorecardSummaryTable(
            ScorecardData data)
        {
            ...
        }
```

The purpose of **GetScorecardSummaryTable** is to aggregate the list of KPIs
across all of the various client sites and determine the status to show for each
time period (based on the value specified for each site). For example, if the
status of any site is "red", then the rollup status is "red." Similarly, if none
of the sites are "red" and the status of any site is "yellow", then the rollup
status is "yellow." Only when the KPI status is "green" for all of the sites
would we want to show "green" in the summary view.

Now you can see how the **ScorecardService** class encapsulates the business
rules we need to implement -- rather than having these scattered throughout the
presentation layer.

You might be wondering why the **GetScorecardSummaryTable** returns a generic
DataTable instead of a typed DataTable. After all, having just proclaimed my
love for typed DataSets, why wouldn't I use Visual Studio to add another table
(e.g. **ScorecardSummary**) to **ScorecardData** and populate an instance of
that table instead of using a generic DataTable?

The problem is that we don't know at design-time what the columns in the summary
table will be. We know the first column in the summary table will contain the
KPI names. However, each additional column in the table corresponds to a time
period specified in the underlying data.

To help visualize this, consider the following example summary view for a KPI
scorecard:

{{< table class="small" caption="Key Performance Indicators (Summary)" >}}

| KPI | 2009 Q3 | 2009 Q4 | 2010 Q1 |
| --- | --- | --- | --- |
| Cycle Time | ![Exceeds](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-0-16x16.gif) | ![Meets](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-1-16x16.gif) | ![Does Not Meet](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-2-16x16.gif) |
| Utilization | ![Exceeds](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-0-16x16.gif) | ![Does Not Meet](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-2-16x16.gif) | ![Meets](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-1-16x16.gif) |
| Rejection Rate |  | ![Does Not Meet](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-2-16x16.gif) | ![Exceeds](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/kpidefault-0-16x16.gif) |

{{< /table >}}

While this example shows three periods of data, there might be cases where fewer
columns are displayed (and perhaps other scenarios where more columns are
displayed). The point is, the presentation layer should simply display whatever
the service layer returns as the "summary table" based on the specified
scorecard data (applying any presentational aspects as necessary -- such as
replacing KPI status values with corresponding `<img>` elements).

When the user selects a KPI in the summary view, we want to display more
information in a detail view on the right side of the page. Consequently, we add
another method to the **ScorecardService** class:

```
        public static DataTable GetScorecardDetailTable(
            ScorecardData data,
            string selectedKpiName)
        {
            ...
        }
```

Similar to the **GetScorecardSummaryTable** method, the
**GetScorecardDetailTable** method returns a generic DataTable that will
subsequently be rendered by the presentation tier (i.e. the ASP.NET user
control). Perhaps something like this:

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

Assuming you are proficient in ADO.NET, it shouldn't take you very long to
implement the **GetScorecardSummaryTable** and **GetScorecardDetailTable**
methods. [In fact, if you are like me, you'll probably spend more time working
on the ASCX control, for example to get the "Thresholds" label to appear above
the corresponding columns in the detail table. (I eventually punted the Telerik
RadGrid control I originally intended to use and ended up replacing it with a
simple GridView control, just so I could easily add a second header row with the
corresponding `rowspan` and `colspan` attributes.)]

So far, we've addressed the following basic scenarios:

- Retrieve a typed DataSet containing the scorecard data for a given list of
  sites.
- Generate a summary table and display it on the page.
- When the user selects a KPI in the summary table, generate a detail table and
  display it on the page.

We've kept the presentation layer "thin" (i.e. minimized the code in
KpiScorecard.ascx) by delegating most of the work to a class in the services
layer (**ScorecardService**).

To make things more interesting -- and move beyond elementary .NET development
-- let's add a new scenario. Suppose that we need to support the ability to
filter by site. In other words, KpiScorecard.ascx will initially show the
scorecard data for all of the sites, but provide a dropdown list to allow the
user to quickly filter the information to, say, the Denver site.

Note that the generic DataTable returned from the **GetScorecardSummaryTable**
method doesn't contain any information about sites (it is, after all, a summary
of the scorecard data across all sites). Consequently, we have to provide a way
to filter on a specific site when generating the scorecard summary table. In
other words, we only want the summary table to include the list of KPIs for a
specific site (and the corresponding KPI status values from various time periods
for that site).

To address this new scenario, we could just add a
<var>selectedClientSiteId</var> parameter to **GetScorecardSummaryTable** that,
if specified (i.e. is not null or empty), is subsequently used to limit the rows
that are added to the summary table. In other words, we could refactor the
**GetScorecardSummaryTable** method to add an overload, as follows:

```
        public static DataTable GetScorecardSummaryTable(
            ScorecardData data)
        {
            return GetScorecardSummaryTable(data, null);
        }

        public static DataTable GetScorecardSummaryTable(
            ScorecardData data,
            string selectedClientSiteId)
        {
            ...

            // If selectedClientSiteId is specified, then filter the scorecard
            // items accordingly
            ...
        }
```

To be honest, I've used this approach in the past, and while it is relatively
easy to make the corresponding changes when refactoring, it's not exactly
*robust*.

For example, it's quite possible that a future scenario would force us to add
yet another overload to the method. To understand why, suppose that instead of,
or in addition to, filtering by a site, we need to provide the ability to filter
on a specific time period. Consequently, we could refactor again to add a
<var>selectedPeriod</var> parameter. Again, this approach is certainly feasible
and, indeed, very straightforward in terms of implementation.

Another approach would be to simply add a generic <var>filterExpression</var>
parameter and allow the caller to specify how the data should be filtered by
specifying something like

> `"ClientSiteId = 'S12345' AND Period = '2010 Q1'"`

However, I've never been fond of the generic "filter expression" approach,
primarily because you have to write a lot of code to support the ability to
specify columns from different tables within the DataSet. It's doable -- and it
does limit the number of method overloads that you would end up with -- but it's
a lot of work.

What if, instead of refactoring the original **GetScorecardSummaryTable** method
to add a <var>selectedClientSiteId</var> parameter, we instead refactor to add
an overload that takes an arbitrary list of rows from the **ScorecardItem**
table? In other words, instead of relying on various method overloads to support
filtering of the scorecard data, we instead refactor to allow the caller to
filter the data and simply operate on the (potentially) filtered rows instead of
all rows in the **ScorecardItem** table.

Note that the most generic way of specifying a list of scorecard items is
`IEnumerable<ScorecardData.ScorecardItemRow>`.

In order to preserve the existing functionality -- and thus avoid having to
change any existing unit tests -- we keep the original method, but simply have
it call the new overload (passing in all rows from the **ScorecardItem** table):

```
        public static DataTable GetScorecardSummaryTable(
            ScorecardData data)
        {
            if (data == null)
            {
                throw new ArgumentNullException("data");
            }

            return GetScorecardSummaryTable(data.ScorecardItem);
        }

        public static DataTable GetScorecardSummaryTable(
            IEnumerable<ScorecardData.ScorecardItemRow> scorecardItems)
        {
            ...
        }
```

This turns out to be incredibly powerful because, in the presentation layer, we
can now support a variety of ways to filter the list of scorecard items (without
having to change the service layer).

In the user control (KpiScorecard.ascx), all we need to do is retrieve the
scorecard data by providing a list of sites (in other words, get an instance of
the **ScorecardData** DataSet), and subsequently determine which items to show:

```
        private void UpdateScorecardSummaryView()
        {
            Debug.Assert(scorecardData != null);

            IEnumerable<ScorecardData.ScorecardItemRow> scorecardItems =
                GetScorecardItemsToShowInKpiSummary();

            DataTable summaryTable = ScorecardService.GetScorecardSummaryTable(
                scorecardItems);

            ScorecardSummaryView.DataSource = summaryTable;
            ScorecardSummaryView.DataBind();
        }
```

For the sake of simplicity, assume that we currently only need to support the
ability to filter by site. The **GetScorecardItemsToShowInKpiSummary** method in
the KpiScorecard.ascx file is implemented as follows:

```
        private IEnumerable<ScorecardData.ScorecardItemRow>
            GetScorecardItemsToShowInKpiSummary()
        {
            Debug.Assert(scorecardData != null);

            if (string.IsNullOrEmpty(siteList.SelectedSiteId) == true)
            {
                return scorecardData.ScorecardItem;
            }
            else
            {
                var scorecardItems =
                    from scorecardItem in scorecardData.ScorecardItem
                    where scorecardItem.ClientSiteId == siteList.SelectedSiteId
                    select scorecardItem;

                return scorecardItems;
            }
        }
```

If we needed to support additional filtering in the future, we could simply
modify the LINQ to DataSet query expression shown above and everything else
would "just work."

Based on the previous change to **GetScorecardSummaryTable** (to support the
ability to provide a filtered list of scorecard items instead of all scorecard
items), we'll need to make a corresponding change to **GetScorecardDetailTable**
(to ensure that only the details for the *filtered* scorecard items appear in
the detail table).

Note that when refactoring code to add overloads that accept an `IEnumerable`
parameter, you'll want to ensure the functionality of the original method
remains the same.

For example, suppose the original **GetScorecardDetailTable** method used a
DataView to filter the **ScorecardItems** table on the specified KPI name:

```
        public static DataTable GetScorecardDetailTable(
            ScorecardData data,
            string selectedKpiName)
        {
            if (data == null)
            {
                throw new ArgumentNullException("data");
            }
            else if (selectedKpiName == null)
            {
                throw new ArgumentNullException("selectedKpiName");
            }
            else if (string.IsNullOrEmpty(selectedKpiName) == true)
            {
                throw new ArgumentException(
                    "The name of the selected KPI must be specified.",
                    "selectedKpiName");
            }

            DataTable detailTable = new DataTable();
            ...

            DataView view = new DataView(
                data.ScorecardItem);

            view.RowFilter = string.Format(
                CultureInfo.InvariantCulture,
                "KpiName = '{0}'",
                selectedKpiName);

            foreach (DataRowView row in view)
            {
                ...
            }

            return detailTable;
        }
```

Note that this implementation inherently enforces the rule that all scorecard
items included in the detail view refer to the same KPI (because of the
RowFilter specified on the DataView). Consequently, when adding an overload for
**GetScorecardDetailTable** to accept an `IEnumerable` parameter, we have a
couple of choices.

We could simply add an overload that accepts an `IEnumerable` parameter in
addition to the original <var>selectedKpiName</var> parameter:

```
        public static DataTable GetScorecardDetailTable(
            IEnumerable<ScorecardData.ScorecardItemRow> scorecardItems,
            string selectedKpiName)
        {
           ...
        }
```

Note that with this approach, we would need to filter the list of scorecard
items to ensure that only items referring to the specified KPI are included in
the detail table (in order to preserve the behavior of the original method).
While not terribly inefficient, it does seem somewhat wasteful to have to filter
the list of scorecard items a second time.

To avoid this issue altogether, we can instead choose to eliminate the
<var>selectedKpiName</var> parameter altogether on the method overload that
takes an `IEnumerable` parameter (and assume instead that all of the scorecard
items refer to the same KPI). Note that we need to add some code to validate the
new assumption:

```
        public static DataTable GetScorecardDetailTable(
            IEnumerable<ScorecardData.ScorecardItemRow> scorecardItems)
        {
            if (scorecardItems == null)
            {
                throw new ArgumentNullException("scorecardItems");
            }

            string selectedKpiName = null;

            foreach (ScorecardData.ScorecardItemRow scorecardItem in
                scorecardItems)
            {
                if (selectedKpiName == null)
                {
                    selectedKpiName = scorecardItem.KpiName;
                }
                else if (string.Compare(
                    scorecardItem.KpiName,
                    selectedKpiName,
                    StringComparison.OrdinalIgnoreCase) != 0)
                {
                    throw new ArgumentException(
                        "All scorecard items must refer to the same KPI.",
                        "scorecardItems");
                }
            }

            ...
        }
```

To reduce the amount of code in the method (and make it easier to understand and
maintain), we should refactor the code that enforces the assumption into a
separate method:

```
        public static DataTable GetScorecardDetailTable(
            IEnumerable<ScorecardData.ScorecardItemRow> scorecardItems)
        {
            if (scorecardItems == null)
            {
                throw new ArgumentNullException("scorecardItems");
            }

            string selectedKpiName = GetDistinctKpiName(scorecardItems);

            ...
        }

        private static string GetDistinctKpiName(
            IEnumerable<ScorecardData.ScorecardItemRow> scorecardItems)
        {
            Debug.Assert(scorecardItems != null);

            string selectedKpiName = null;

            foreach (ScorecardData.ScorecardItemRow scorecardItem in
                scorecardItems)
            {
                if (selectedKpiName == null)
                {
                    selectedKpiName = scorecardItem.KpiName;
                }
                else if (string.Compare(
                    scorecardItem.KpiName,
                    selectedKpiName,
                    StringComparison.OrdinalIgnoreCase) != 0)
                {
                    throw new ArgumentException(
                        "All scorecard items must refer to the same KPI.",
                        "scorecardItems");
                }
            }

            return selectedKpiName;
        }
```

> **Note**
>
> You might have chosen to use LINQ instead of the `foreach` loop to verify the
> list of scorecard items all refer to the same KPI and return it. Just be sure
> you account for scenarios such as when the KPI names specified by the
> scorecard items differ only by case, or when the list of scorecard items is
> empty.

The original version of the **GetScorecardDetailTable** then simply needs to
filter the scorecard items based on the specified KPI name (which is really easy
using a LINQ query expression) and defer the rest of the work to the new
overload of the method (shown above):

```
        public static DataTable GetScorecardDetailTable(
            ScorecardData data,
            string selectedKpiName)
        {
            if (data == null)
            {
                throw new ArgumentNullException("data");
            }
            else if (selectedKpiName == null)
            {
                throw new ArgumentNullException("selectedKpiName");
            }
            else if (string.IsNullOrEmpty(selectedKpiName) == true)
            {
                throw new ArgumentException(
                    "The name of the selected KPI must be specified.",
                    "selectedKpiName");
            }

            var scorecardItems =
                from scorecardItem in data.ScorecardItem
                where scorecardItem.KpiName == selectedKpiName
                select scorecardItem;

            return GetScorecardDetailTable(scorecardItems);
        }
```

> **Important**
>
> Note that by replacing the original approach of filtering the scorecard items
> (using a DataView) with a LINQ query expression, we've made a substantial
> improvement in the code. For example, if we were to rename the KpiName column
> in the typed DataSet without making the corresponding change to the filtering
> code, we would get a compile-time error instead of a run-time error. When
> using the original DataView approach, a similar mistake would not be caught at
> compile-time (because the column name is embedded in the RowFilter string
> value).

Let's wrap this up with a few key points:

- You can greatly reduce the amount of code in the presentation layer by using a
  typed DataSet in combination with a corresponding "service" class (that is
  responsible for abstracting the details of populating the data, and also
  provides other methods for manipulating the data).
- Get your core scenarios working and then refactor your code to address
  additional scenarios (such as filtering the data).
- To support robust filtering of your data, add method overloads that accept an
  `IEnumerable<>` list of items (thus allowing the caller to filter the data in
  a variety of ways without requiring a change to the services layer).
- Using LINQ with typed DataSets makes it very easy to filter your data (while
  also catching potential problems at compile-time instead of at run-time).
