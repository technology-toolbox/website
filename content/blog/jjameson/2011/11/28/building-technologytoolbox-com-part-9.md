---
title: "Using the Entity Framework and LINQ to list the most recent posts from Subtext (a.k.a. Building TechnologyToolbox.com, part 9)"
date: 2011-11-28T09:22:16-07:00
excerpt: "In a previous post, I mentioned how the new Technology Toolbox home page highlights 
the most recent blog posts from Subtext. In this post, I'll show you how easy this feature was to develop -- thanks to the Entity Framework and LINQ."
draft: true
categories: ["Development", "My System"]
tags: ["Core 
			Development", "Subtext", "My System", "Web Development"]
---

In
[a previous post](/blog/jjameson/2011/10/18/introducing-technologytoolbox-com), I mentioned how the new Technology Toolbox home page highlights
the most recent blog posts, as shown below.

![Technology Toolbox home page](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Home.png)

    	Figure 1: Technology Toolbox home page

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Home.png)

The content rendered in the **Most Recent Posts** section is
generated using an ASP.NET user control, as illustrated in the corresponding
page layout.

![Technology Toolbox home page (page layout)](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Home%20(Page%20Layout).png)

    	Figure 2: Technology Toolbox home page (page layout)

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Home%20%28Page%20Layout%29.png)

In that post, I also showed how the Technology Toolbox site is comprised
of two Visual Studio solutions that are merged together during the deployment
process. Requests for URLs under **/blog** are handled by Subtext.
All other requests (such as the Technology Toolbox home page) are handled by
the "Caelum" solution.

![Solution architecture](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Solution-Architecture.jpg)

    	Figure 3: Solution architecture

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Solution-Architecture.jpg)

Let's start with the data access layer and work our way up to the presentation
layer (i.e. the ASP.NET user control).

### Data Access Layer

Accessing information stored in databases from .NET applications has always
been fairly easy -- thanks to .NET Framework classes like DataSet, DataReader,
etc. However, in recent years it became even easier with the introduction of
the Entity Framework.

While Subtext already has a data access layer, I chose to create a new DAL
in the Caelum solution in order to leverage the Entity Framework. I'm typically
not a fan of duplicating functionality like this, but in this case it seemed
logical -- especially when you consider how easy it is to get started with the
Entity Framework and how little custom code it subsequently requires to read
or update data.

To create the new DAL, I added a new C# library project (**Data.csproj**)
to the solution in order to isolate the DAL in a separate assembly (**TechnologyToolbox.Caelum.Data.dll**).
Then I added a new **ADO.NET Entity Data Model** to the project
(**Caelum.edmx**).

Since Subtext stores blog posts in the **subtext\_Content** table,
I added that table to the model. Next I updated the properties to change the
entity names as follows:

- **Entity Set Name:** Entries
- **Name:** Entry

These changes make the LINQ queries much easier to read. For example, the
following LINQ query can be used to retrieve the three most recent blog posts:

```
using (CaelumEntities context = new CaelumEntities())
    {
        var q = (from entry in context.Entries
                 orderby entry.DateSyndicated descending
                 select entry).Take(3);

        // TODO: Process the query results (e.g. data bind to a control)
    }
```

Note that **DateSyndicated** represents the date/time a blog
post was published, so ordering the results by this property (in descending
order) and then using the
[**Take()**](http://msdn.microsoft.com/en-us/library/bb503062.aspx)
method (to limit the results to three) yields the desired items for the
**Most Recent Posts** section.

Once I had the minimal "plumbing" necessary to retrieve the list of recent
posts from the Subtext database, I turned my attention to displaying the results.

### RecentPosts.ascx

As noted in
[part 5 of this series](/blog/jjameson/2011/11/09/building-technologytoolbox-com-part-5), the Technology Toolbox site currently uses the
[hAtom 0.1 microformat](http://microformats.org/wiki/hatom) to render
semantic HTML for blog posts.

Here is the HTML structure for the **Most Recent Posts** section
on the site home page (corresponding to the screenshot in Figure 1):

```
<div class="hfeed posts-recent">
    <h2>Most Recent Posts</h2>
    <div class="hentry">
        <h3 class="entry-title">
            <a href="/blog/jjameson/archive/2011/10/17/introducing-technologytoolbox-com.aspx"
                rel="bookmark">Introducing TechnologyToolbox.com</a></h3>
        <ul class="post-info">
            <li class="published">
                <span class="label">Published </span>
                <span class="value">October 17, 2011</span>
                <span class="label"> at </span>
                <span class="value">11:51 AM</span>
            </li>
            <li class="vcard author">
                by <span class="fn">Jeremy Jameson</span>
            </li>
            <li class="comments none">
                <a href="/blog/jjameson/archive/2011/10/17/introducing-technologytoolbox-com.aspx#postComments">
                <span class="label">Comments: </span>
                <span class="value count">0</span></a>
            </li>
        </ul>
        <div class="entry-summary">
            <p>In this inaugural post for my new blog location, I'll introduce
                various features of the new Technology Toolbox website and
                provide a high-level overview of the underlying architecture.
                Subsequent posts will cover different aspects of the site in
                greater detail.</p>
        </div>
    </div>
    <div class="hentry">
        <h3 class="entry-title">
            <a href="/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx"
                rel="bookmark">
                Last Day with Microsoft</a></h3>
        <ul class="post-info">
            <li class="published">
                <span class="label">Published </span>
                <span class="value">September 2, 2011</span>
                <span class="label"> at </span>
                <span class="value">3:43 PM</span>
            </li>
            <li class="vcard author">by <span class="fn">
                Jeremy Jameson</span></li>
            <li class="comments">
                <a href="/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx#postComments">
                    <span class="label">Comments: </span>
                    <span class="value count">11</span>
                </a>
            </li>
        </ul>
        <div class="entry-summary">
            ...
        </div>
    </div>
    <div class="hentry">
        <h3 class="entry-title">
            <a href="/blog/jjameson/archive/2011/09/02/new-blog-location-http-www-technologytoolbox-com-blog-jjameson.aspx"
                rel="bookmark">...</a></h3>
        <ul class="post-info">
            ...
        </ul>
        <div class="entry-summary">
            ...
        </div>
    </div>
</div>
```

Once I have defined the semantic markup for a feature (usually via
[a static HTML prototype](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-3)), I typically copy the sample HTML into a user control
and then start replacing the sample content with ASP.NET controls to render
the dynamic content.

For the **Most Recent Posts** section on the site home page,
I added a new user control to the project (**RecentPosts.ascx**),
copied the sample HTML above into the user control, and then added an instance
of the control to the site home page.

Next I replaced the static `<div  class="hentry">` elements
with an ASP.NET **Repeater** control:

```
<div class="hfeed posts-recent">
    <h2>Most Recent Posts</h2>    
    <asp:Repeater runat="server" ID="PostList">
        <ItemTemplate>
            <div class="hentry">
                <h3 class="entry-title">
                    <a href="{TODO: /blog/jjameson/archive/2011/10/17/introducing-technologytoolbox-com.aspx}"
                        rel="bookmark"><%# Eval("Title") %></a></h3>
                <ul class="post-info">
                    <li class="published">
                        <span class="label">Published </span>
                        <span class="value">{TODO: October 17, 2011}</span>
                        <span class="label"> at </span>
                        <span class="value">{TODO: 11:51 AM}</span>
                    </li>
                    <li class="vcard author">
                        by <span class="fn"><%# Eval("Author") %></span>
                    </li>
                    <li class="comments none">
                        <a href="{TODO: /blog/jjameson/archive/2011/10/17/introducing-technologytoolbox-com.aspx}#postComments">
                        <span class="label">Comments: </span>
                        <span class="value count">{TODO: 0}</span></a>
                    </li>
                </ul>
                <div class="entry-summary">
                    <p><%# Eval("Description") %></p>
                </div>
            </div>
        </ItemTemplate>
    </asp:Repeater>
</div>
```

> **Note**
>
> When writing code, I generally prefer to take little steps -- rather than trying to do too much at once. For example, as shown above, I will often add "TODO:" placeholders to indicate where additional work needs to be done in order to replace sample content (such as generating the URL to a blog post).

In the corresponding code-behind for the user control, I added code to retrieve
the list of recent posts and bind the results to the **Repeater** control:

```
using System;
using System.Linq;
using TechnologyToolbox.Caelum.Data;

namespace TechnologyToolbox.Caelum.Website.Controls
{
    /// ...
    public partial class RecentPosts : System.Web.UI.UserControl
    {
        protected void Page_Load(
            object sender,
            EventArgs e)
        {
            using (CaelumEntities context = new CaelumEntities())
            {
                var q = (from entry in context.Entries
                         orderby entry.DateSyndicated descending
                         select entry).Take(3);

                PostList.DataSource = q;
                PostList.DataBind();
            }
        }
    }
}
```

At this point, I ran a quick test to ensure this worked as expected.

In order to generate the URL for a specific blog post, I created a simple
"helper" class with a method that generates the server-relative URL of a blog
post (for example, "/blog/jjameson/archive/2011/10/17/introducing-technologytoolbox-com.aspx"):

```
namespace TechnologyToolbox.Caelum.Website
{
    /// ...
    public static class BlogHelper
    {
        /// ...
        public static Uri GetEntryUrl(
               string entryName,
               DateTime dateSyndicated)
        {
            if (string.IsNullOrEmpty(entryName) == true)
            {
                throw new ArgumentException(
                    "The entry name must be specified.");
            }

            string entryUrl = string.Format(
                CultureInfo.InvariantCulture,
                "/blog/jjameson/archive/{0:yyyy}/{0:MM}/{0:dd}/{1}.aspx",
                dateSyndicated,
                entryName);

            return new Uri(entryUrl, UriKind.Relative);
        }
    }
}
```

> **Important**
>
> The blog path (i.e. /blog/jjameson) is currently hard-coded in this method. While I briefly considered enhancing this to support other scenarios, I decided against it since it currently suits my needs. If and when I ever need to support other blogs on the Technology Toolbox site, I will obviously need to revisit this piece of code.

Then I replaced the placeholders for the post URL with corresponding calls
to the **BlogHelper.GetEntryUrl** method:

```
<div class="hfeed posts-recent">
    <h2>Most Recent Posts</h2>
    <asp:Repeater runat="server" ID="PostList">
        <ItemTemplate>
            <div class="hentry">
                <h3 class="entry-title">
                    <a href="<%# BlogHelper.GetEntryUrl(
                        (string) Eval("EntryName"),
                        (DateTime) Eval("DateSyndicated")) %>"
                        rel="bookmark"><%# Eval("Title") %></a></h3>
                <ul class="post-info">
                    <li class="published">
                        <span class="label">Published </span>
                        <span class="value">{TODO: October 17, 2011}</span>
                        <span class="label"> at </span>
                        <span class="value">{TODO: 11:51 AM}</span>
                    </li>
                    <li class="vcard author">
                        ...
                    </li>
                    <li class="comments none">
                        <asp:HyperLink runat="server" NavigateUrl='<%# string.Format(
                            "{0}#postComments",
                            BlogHelper.GetEntryUrl(
                                (string) Eval("EntryName"),
                                (DateTime) Eval("DateSyndicated"))) %>'>
                          <span class="label">Comments: </span>
                          <span class="value count">{TODO: 0}</span>
                        </asp:HyperLink>                     </li>
                </ul>
                ...
            </div>
        </ItemTemplate>
    </asp:Repeater>
</div>
```

After running another quick test to verify the URLs are generated correctly,
I replaced the sample date/time values for each post with the actual values
specified in **DateSyndicated**. This is simply a matter of formatting
the **DateTime** value as shown below:

```
<div class="hfeed posts-recent">
    <h2>Most Recent Posts</h2>
    <asp:Repeater runat="server" ID="PostList">
        <ItemTemplate>
            <div class="hentry">
                <h3 class="entry-title">
                    ...</h3>
                <ul class="post-info">
                    <li class="published">
                        <span class="label">Published </span>
                        <span class="value"><%# Eval( "DateSyndicated", "{0:MMMM d, yyyy}") %></span>
                        <span class="label"> at </span>
                        <span class="value"><%# Eval( "DateSyndicated", "{0:t}") %></span>
                    </li>
                    ...
                </ul>
                ...
            </div>
        </ItemTemplate>
    </asp:Repeater>
</div>
```

At this point, the feature was almost complete. The only thing left to do
was to replace the placeholder for the number of comments and conditionally
add the `class="none"`
attribute value to the `<li  class="comments">` element. In other words, when there
are no comments for a post, the markup should be `<li  class="comments none">`, but when there is at least one
comment the markup should be `<li  class="comments">`. This makes it very easy to show or
hide the comments icon (and corresponding link) using CSS.

```
<div class="hfeed posts-recent">
    <h2>Most Recent Posts</h2>
    <asp:Repeater runat="server" ID="PostList">
        <ItemTemplate>
            <div class="hentry">
                <h3 class="entry-title">
                    ...</h3>
                <ul class="post-info">
                    <li class="published">
                        ...
                    </li>
                    <li class="vcard author">
                        ...
                    </li>
                    <li class="comments<%# (int)Eval("FeedBackCount") == 0 ? " none" : string.Empty %>">
                        <asp:HyperLink runat="server"
                          NavigateUrl='<%# string.Format(
                            "{0}#postComments",
                            BlogHelper.GetEntryUrl(
                                (string) Eval("EntryName"),
                                (DateTime) Eval("DateSyndicated"))) %>'>
                          <span class="label">Comments: </span>
                          <span class="value count"><%# Eval( "FeedBackCount") %></span>
                        </asp:HyperLink>
                    </li>
                </ul>
                <div class="entry-summary">
                    ...
                </div>
            </div>
        </ItemTemplate>
    </asp:Repeater>
</div>
```

After replacing the last "TODO:" placeholder and running a few more tests
to verify the feature was complete, I turned my attention to performance tuning.

In order to greatly reduce the number of roundtrips to SQL Server, I added
a cache directive to the user control and specified a duration of five minutes
(300 seconds):

```
<%@ OutputCache Duration="300" VaryByParam="None" %>
```

Here is the complete source for RecentPosts.ascx:

```
<%@ Control Language="C#" AutoEventWireup="true"
    CodeBehind="RecentPosts.ascx.cs"
    Inherits="TechnologyToolbox.Caelum.Website.Controls.RecentPosts" %>
<%@ OutputCache Duration="300" VaryByParam="None" %>
<%@ Import Namespace="TechnologyToolbox.Caelum.Website" %>
<div class="hfeed posts-recent">
    <h2>Most Recent Posts</h2>
    <asp:Repeater runat="server" ID="PostList">
        <ItemTemplate>
            <div class="hentry">
                <h3 class="entry-title">
                    <a href="<%# BlogHelper.GetEntryUrl(
                        (string) Eval("EntryName"),
                        (DateTime) Eval("DateSyndicated")) %>"
                       rel="bookmark"><%# Eval("Title") %></a></h3>
                <ul class="post-info">
                    <li class="published">
                        <span class="label">Published </span>
                        <span class="value"><%# Eval(
                            "DateSyndicated",
                            "{0:MMMM d, yyyy}") %></span>
                        <span class="label"> at </span>
                        <span class="value"><%# Eval(
                            "DateSyndicated",
                            "{0:t}") %></span>
                    </li>
                    <li class="vcard author">
                        by <span class="fn"><%# Eval("Author") %></span>
                    </li>
                    <li class="comments<%# (int)Eval("FeedBackCount") == 0 ? " none" : string.Empty %>">
                        <asp:HyperLink runat="server"
                          NavigateUrl='<%# string.Format(
                            "{0}#postComments",
                            BlogHelper.GetEntryUrl(
                                (string) Eval("EntryName"),
                                (DateTime) Eval("DateSyndicated"))) %>'>
                          <span class="label">Comments: </span>
                          <span class="value count"><%# Eval(
                            "FeedBackCount") %></span>
                      </asp:HyperLink>
                    </li>
                </ul>
                <div class="entry-summary">
                    <p><%# Eval("Description") %></p>
                </div>
            </div>
        </ItemTemplate>
    </asp:Repeater>
</div>
```

