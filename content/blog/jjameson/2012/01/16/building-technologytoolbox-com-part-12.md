---
title: "Using jQuery to create an expandable archive list for blog posts (a.k.a. Building TechnologyToolbox.com, part 12)"
date: 2012-01-16T02:43:57-07:00
excerpt: "In my previous post, I briefly mentioned how I use a CSS sprite and jQuery to render the expandable list under the \"Archives\" section on the various blog pages of the Technology Toolbox site. This post details the implementation of that feature."
draft: true
categories: ["Development", "My System"]
tags: ["Subtext", "Web Development"]
---

In
[my previous post](/blog/jjameson/2012/01/15/building-technologytoolbox-com-part-11), I briefly mentioned how I use a CSS sprite and jQuery
to render the expandable list under the **Archives** section on
the various blog pages of the Technology Toolbox site.

![Blog home page](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Blog-Home.png)

    	Figure 1: Blog home page

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Blog-Home.png)

One of my "love/hate" features on
[my old MSDN blog](http://blogs.msdn.com/jjameson/) was the "Archive"
list of links on the right side of each page (showing each month in which I
created a post as well as the number of posts that I created in each of those
months). I "loved" it because this allowed users to easily browse through all
of my old posts, as well as providing some at-a-glance information (such as
when I created my first blog post and which months, for one reason or another,
I authored an unusually large number of posts). I "hated" this feature because
it took up an awful lot of real estate on each blog page.

Consequently, when it came time to create my new blog, I knew I wanted to
provide similar functionality but "minimize" the archive list by default and
allow users to "drill into" the list if they choose to.

If you look "under the covers" at one of the pages on my new blog, you can
see how I started out by defining some semantic markup for the **Archives** section
[using a static HTML prototype](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-3):

```
<div class="posts-archive">
  <h2>Archives</h2>
  <ul>
    <li>2011
      <ul>
        <li><a href="/blog/jjameson/archive/2011/09.aspx">September (2)</a></li>
        <li><a href="/blog/jjameson/archive/2011/08.aspx">August (1)</a></li>
        <li><a href="/blog/jjameson/archive/2011/05.aspx">May (3)</a></li>
        <li><a href="/blog/jjameson/archive/2011/04.aspx">April (10)</a></li>
        <li><a href="/blog/jjameson/archive/2011/03.aspx">March (22)</a></li>
        <li><a href="/blog/jjameson/archive/2011/02.aspx">February (6)</a></li>
        <li><a href="/blog/jjameson/archive/2011/01.aspx">January (2)</a></li>
      </ul>
    </li>
    <li>2010
      <ul>
        <li><a href="/blog/jjameson/archive/2010/12.aspx">December (8)</a></li>
        ...
      </ul>
    </li>
    ...
  </ul>
</div>
```

This HTML structure looks good "naked" (i.e. with CSS disabled) and therefore
passes my basic "sniff" test for semantic HTML. Therefore, I turned my attention
to creating an ASP.NET control to render the HTML based on a query to the underlying
Subtext database.

Since the markup is really simple, I chose to create an ASP.NET Web control
(a.k.a. "server control") instead of a user control (i.e. an ASCX file).

However, before jumping straight into the presentation layer, let's first
review the "plumbing" used to query the necessary data from the Subtext database.

### Data Access Layer

If you've been following along in this series, then you know that by this
point in the development timeline I had already created an ADO.NET Entity Data
Model for accessing data from Subtext.

![Entity Data Model](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Entity-Model-Caelum-Step-2.png)

    	Figure 2: Entity Data Model

Consequently I spent a few minutes translating the SQL query in my head for
grouping posts by year/month into the equivalent LINQ query against the data
model. This is what I ended up:

```
var q = from entry in context.Entries
            group entry by entry.DateSyndicated.Value.Year
                into YearGroups
            orderby YearGroups.Key descending
            select new
            {
                Year = YearGroups.Key,
                MonthGroups =
                    from yearGroup in YearGroups
                    group yearGroup
                        by yearGroup.DateSyndicated.Value.Month
                        into MonthGroups
                    orderby MonthGroups.Key descending
                    select new
                    {
                        Month = MonthGroups.Key,
                        Count = MonthGroups.Count()
                    }
            };
```

It might look a little nasty at first glance, so you may want think of it
as a simple "GROUP BY DATEPART(YEAR, DateSyndicated)" T-SQL query with a
[correlated subquery](http://en.wikipedia.org/wiki/Correlated_subquery)
(to group by month and count the number of rows for each month).

> **Tip**
>
> If you are working with LINQ these days, and you haven't already discovered [LINQPad](http://www.linqpad.net/), I highly recommend checking it out. It makes it much quicker to iteratively build LINQ queries like the one shown above.

![LINQPad](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_LINQPad.png)

    	Figure 3: LINQPad

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_LINQPad.png)

### PostArchiveList Control

With the LINQ query ready, I turned my attention to displaying the results.
As I mentioned before, I chose to create a server control in this case (rather
than a user control) since the HTML that needs to be emitted is relatively simple.

Since I prefer to develop iteratively (moving rapidly in a series of small
steps), I started by setting the CSS class for the control and overriding the
**[TagKey](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.webcontrol.tagkey.aspx)** property (since it defaults to `<span>`
-- and I want this control to emit a `<div>` instead). I also
added the "Archives" heading as well as a **Literal** control to
serve as a placeholder for the list(s) that will be generated from the query
results.

```
namespace TechnologyToolbox.Caelum.Website.Controls
{
    public class PostArchiveList : WebControl
    {
        Literal list;

        public PostArchiveList()
        {
            this.CssClass = "posts-archive";
        }

        protected override HtmlTextWriterTag TagKey
        {
            get { return HtmlTextWriterTag.Div; }
        }

        protected override void CreateChildControls()
        {
            base.CreateChildControls();
            
            this.Controls.Add(new LiteralControl("<h2>Archives</h2>"));

            list = new Literal();
            this.Controls.Add(list);
        }

        protected override void OnPreRender(
            EventArgs e)
        {
            base.OnPreRender(e);

                StringBuilder buffer = new StringBuilder();
                buffer.Append("<ul>");

                buffer.Append("<li>TODO: Show results from query</li>");

                buffer.Append("</ul>");

                list.Text = buffer.ToString();
            }
        }
    }
}
```

Once I confirmed the basic structure of the HTML was being emitted as desired,
I added the code to generate the series of nested lists for each year/month
in the query results:

```
protected override void OnPreRender(
            EventArgs e)
        {
            base.OnPreRender(e);

            using (CaelumEntities context = new CaelumEntities())
            {
                var q = from entry in context.Entries
                        group entry by entry.DateSyndicated.Value.Year
                            into YearGroups
                        orderby YearGroups.Key descending
                        select new
                        {
                            Year = YearGroups.Key,
                            MonthGroups =
                                from yearGroup in YearGroups
                                group yearGroup
                                    by yearGroup.DateSyndicated.Value.Month
                                    into MonthGroups
                                orderby MonthGroups.Key descending
                                select new
                                {
                                    Month = MonthGroups.Key,
                                    Count = MonthGroups.Count()
                                }
                        };

                StringBuilder buffer = new StringBuilder();
                buffer.Append("<ul>");

                foreach (var result in q)
                {
                    buffer.AppendFormat(
                        CultureInfo.CurrentCulture,
                        "<li>{0}",
                        result.Year);

                    buffer.Append("<ul>");

                    foreach (var monthGroup in result.MonthGroups)
                    {
                        string monthName = GetMonthName(monthGroup.Month);

                        buffer.AppendFormat(
                            CultureInfo.CurrentCulture,
"<li><a href='/blog/jjameson/archive/{0}/{1:D2}.aspx'>{2} ({3})</a></li>",
                            result.Year,
                            monthGroup.Month,
                            monthName,
                            monthGroup.Count);
                    }

                    buffer.Append("</ul>");
                    buffer.Append("</li>");
                }

                buffer.Append("</ul>");

                list.Text = buffer.ToString();
            }
        }
```

The **GetMonthName** method is, as expected, rather trivial:

```
private static string GetMonthName(
            int month)
        {
            DateTime date = new DateTime(2011, month, 1);

            return date.ToString("MMMM", CultureInfo.CurrentCulture);
        }
```

> **Note**
>
> As I've noted before, there are some assumptions in this code that
> I currently consider to be "good enough" (such as the hardcoded path
> to the blog -- i.e.  "/blog/jjameson/archive"). If the time comes
> when this code needs to support other scenarios, then -- and probably
> *only* then -- I'll put the effort into making it more robust.
>
> Also note that there *may* be a way to combine the **GetMonthName** logic into the actual LINQ query, but I couldn't
> find one. Keep in mind that the results need to be ordered chronologically
> in reverse order (for example, for each year the "November" results
> need to come before "January" results).

At this point, the control is nearly identical to the feature on my old MSDN
blog (which is to say that it serves the basic purpose, but it consumes too
much real estate on the page by default). All I needed was a little JavaScript
to collapse the nested lists (in other words, showing only the years) by default
and allow users to expand them as desired.

### jQuery Plugin to Expand/Collapse Lists

I did a quick Internet search for some existing jQuery that I could use instead
of writing my own and I quickly discovered the following:

<cite>Simple jQuery Expand/Collapse Unordered Lists</cite>
[http://www.fluidbyte.net/simple-jquery-expandcollapse-unordered-lists](http://www.fluidbyte.net/simple-jquery-expandcollapse-unordered-lists)

Consequently, I downloaded the plugin and added it to my static HTML prototype
in order try it out and verify everything worked as expected.

However, I discovered a couple of issues with that code (written by Kent
Safranski):

- It didn't work with my markup because it assumes the parent `<ul>`
  elements have ID attributes. For example:
  
  ```
  jQuery.fn.jqcollapse = function (o) {
  
      ...
  
      $(this).each(function () {
  
          var e = $(this).attr('id');
  
          $('#' + e + ' li > ul').each(function (i) {
             ...
          });
  
          ...
          $('#' + e + ' ul').hide();
      });
  };
  ```
  
  Now, I certainly don't consider myself a jQuery expert but, generally speaking,
  this doesn't seem like very good practice when writing jQuery plugins.

- It doesn't maintain chainability (which is generally recommended when
  [writing jQuery plugins](http://docs.jquery.com/Plugins/Authoring)).

Don't get me wrong...my intent is not to bash Kent's code. On the contrary,
with just a little bit of work, I was able to tweak his code into the following:

```
// collapseList() was originally based on the following jQuery plug-in:
//
//   http://www.fluidbyte.net/simple-jquery-expandcollapse-unordered-lists
(function ($) {
    $.fn.collapseList = function (options) {
        // Create some defaults, extending them with any options provided
        var settings = $.extend({
                slide: true,
                speed: 300,
                easing: ''
            },
            options);

        return this.each(function () {
            $(this).find('li>ul').each(function () {
                var parentListItem = $(this).parent('li');
                var childList = $(this).remove();

                if (parentListItem.children('a').length == 0) {
                    parentListItem.wrapInner('<a/>');
                }

                parentListItem.addClass('expandable');

                parentListItem.find('a').css('cursor', 'pointer').click(
                    function () {
                        if (settings.slide == true) {
                            childList.slideToggle(
                                settings.speed,
                                settings.easing);
                        }
                        else {
                            childList.toggle();
                        }

                        parentListItem.toggleClass('expandable');
                        parentListItem.toggleClass('expanded');
                    });

                parentListItem.append(childList);

                $(this).hide();
            });
        });

    };
})(jQuery);
```

Note that I toggle a couple of CSS classes (specifically, `expanded` and `expandable`) on the list item that
was clicked and use a corresponding CSS sprite to show the "plus" or "minus"
icon next to the list item.

![list-item-sprites-1.0.png](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_list-item-sprites-1.0.png)

    	Figure 4: list-item-sprite-1.0.png

As I noted in my previous post, if separate image files are used for the
"plus" and "minus" icons, users experience a subtle flashing effect the first
time they expand one of the years (because the "minus" icon has to be downloaded).
By using a CSS sprite instead, the user experience is improved (albeit a slight
improvement) because the "minus" icon has already been downloaded. Simply tweaking
the x and y positions toggles which icon is displayed.

Here are the corresponding CSS rules:

```
ul li.expandable {
  background: url('Images/list-item-sprites-1.0.png') no-repeat -100px -246px;
  padding-left: 15px;
}
ul li.expanded {
  background: url('Images/list-item-sprites-1.0.png') no-repeat -150px -196px;
  padding-left: 15px;
}
```

Thanks to the great head start provided by Kent's code, I was able to complete
the expand/collapse feature in less than an hour.

After adding the script to the **PostArchiveList** control (and
adding a **[PartialCachingAttribute](http://msdn.microsoft.com/en-us/library/system.web.ui.partialcachingattribute.aspx)** to improve performance by minimizing the
number of database calls), I ended up with the following:

```
using System;
using System.Globalization;
using System.Linq;
using System.Text;
using System.Web.UI;
using System.Web.UI.WebControls;
using TechnologyToolbox.Caelum.Data;

namespace TechnologyToolbox.Caelum.Website.Controls
{
    [PartialCaching(300)]
    public class PostArchiveList : WebControl
    {
        Literal list;

        public PostArchiveList()
        {
            this.CssClass = "posts-archive";
        }

        protected override HtmlTextWriterTag TagKey
        {
            get { return HtmlTextWriterTag.Div; }
        }

        protected override void CreateChildControls()
        {
            base.CreateChildControls();
            
            this.Controls.Add(new LiteralControl("<h2>Archives</h2>"));

            list = new Literal();
            this.Controls.Add(list);
        }

        private static string GetMonthName(
            int month)
        {
            DateTime date = new DateTime(2011, month, 1);

            return date.ToString("MMMM", CultureInfo.CurrentCulture);
        }

        protected override void OnPreRender(
            EventArgs e)
        {
            base.OnPreRender(e);

            using (CaelumEntities context = new CaelumEntities())
            {
                var q = from entry in context.Entries
                        group entry by entry.DateSyndicated.Value.Year
                            into YearGroups
                        orderby YearGroups.Key descending
                        select new
                        {
                            Year = YearGroups.Key,
                            MonthGroups =
                                from yearGroup in YearGroups
                                group yearGroup
                                    by yearGroup.DateSyndicated.Value.Month
                                    into MonthGroups
                                orderby MonthGroups.Key descending
                                select new
                                {
                                    Month = MonthGroups.Key,
                                    Count = MonthGroups.Count()
                                }
                        };

                StringBuilder buffer = new StringBuilder();
                buffer.Append("<ul>");

                foreach (var result in q)
                {
                    buffer.AppendFormat(
                        CultureInfo.CurrentCulture,
                        "<li>{0}",
                        result.Year);

                    buffer.Append("<ul>");

                    foreach (var monthGroup in result.MonthGroups)
                    {
                        string monthName = GetMonthName(monthGroup.Month);

                        buffer.AppendFormat(
                            CultureInfo.CurrentCulture,
"<li><a href='/blog/jjameson/archive/{0}/{1:D2}.aspx'>{2} ({3})</a></li>",
                            result.Year,
                            monthGroup.Month,
                            monthName,
                            monthGroup.Count);
                    }

                    buffer.Append("</ul>");
                    buffer.Append("</li>");
                }

                buffer.Append("</ul>");

                buffer.Append("<script type='text/javascript'>");
                buffer.Append("$('.posts-archive>ul').collapseList();");
                buffer.Append("</script>");

                list.Text = buffer.ToString();
            }
        }
    }
}
```

