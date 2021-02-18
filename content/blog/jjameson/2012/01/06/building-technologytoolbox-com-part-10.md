---
title: "Using the Entity Framework and LINQ to list the most popular posts from Subtext (a.k.a. Building TechnologyToolbox.com, part 10)"
date: 2012-01-06T03:04:06-07:00
excerpt: "In my previous post, I shared the inner workings of the \"Most Recent Posts\" section on the Technology Toolbox home page. In this post, I'll show you how I built on that foundation to generate the content for the \"Most Popular Posts\" section."
aliases: ["/blog/jjameson/archive/2012/01/06/building-technologytoolbox-com-part-10.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["Core 
			Development", "My System", "Subtext", "Web Development"]
---

In
[my previous post](/blog/jjameson/2011/11/28/building-technologytoolbox-com-part-9), I shared the inner workings of the **Most Recent
Posts** section on the Technology Toolbox home page. In this post, I'll
show you how I built on that foundation to generate the content for the
**Most Popular Posts** section.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Home.png"
alt="Technology Toolbox home page"
height="600"
width="538"
title="Figure 1: Technology Toolbox home page" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Home.png)

Similar to the **Most Recent Posts** section, the **Most
Popular Posts** section is generated using an ASP.NET user control, as
illustrated in the corresponding page layout.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Home%20(Page%20Layout).png"
alt="Technology Toolbox home page (page layout)"
height="600"
width="536"
title="Figure 2: Technology Toolbox home page (page layout)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Home%20%28Page%20Layout%29.png)

Note that in the Subtext database, blog posts are stored in the **subtext\_Content**
table and the statistics about the number of views for each blog post are stored
in the **subtext\_EntryViewCount** table. Therefore, the first step
in developing the **Most Popular Posts** feature for the home page
was to expand the entity model.

### Updating the Data Access Layer

I completed the following steps to update the entity model:

1. In Visual Studio, open the entity data model (**Caelum.edmx**).
2. In the Entity Data Model Designer, right-click the background and then
   click **Update Model from Database...**
3. In the **Update Wizard**window:
   1. If necessary, on the **Choose Your Data Connection**
      step, ensure the Subtext database is selected and then click **Next**.
   2. On the **Choose Your Database Objects** step, in the
      **Add** tab, expand **Tables**, click the
      check box for **subtext\_EntryViewCount** (and **subtext\_Content**
      if this table was not previously added to the model), and then click
      **Finish**.
4. For the **subtext\_Content**table, ensure the properties
   are set as follows:
   - **E****ntity
     Set Name:** Entries
   - **N****ame:** Entry
5. For the **subtext\_EntryViewCount**table, ensure the properties
   are set as follows:
   - **E****ntity
     Set Name:** EntryViewCounts
   - **N****ame:** EntryViewCount

At this point, the model should resemble the following:

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Entity-Model-Caelum-Step-2.png"
alt="Entity Data Model"
height="400"
width="431"
title="Figure 3: Entity Data Model" >}}

With the updated model, the following LINQ query can be used to retrieve
the top 10 most popular blog posts:

```
using (CaelumEntities context = new CaelumEntities())
        {
            var q = (from entry in context.Entries
                    join views in context.EntryViewCounts
                        on entry.ID equals views.EntryID
                    orderby (views.WebCount * 15)
                            + (views.AggCount * 10) descending
                    select entry).Take(10);

        }
```

In Subtext, **WebCount** represents the number of times a blog
post has been viewed through the website, and **AggCount** represents
"aggregated" views (e.g. posts viewed by RSS subscribers). Note that I chose
to assign "weighting" factors to Web views vs. RSS views. This is similar to
the formula used in the out-of-the-box **subtext\_GetPopularPosts** stored procedure (although, unlike that sproc, I chose *not*
to include the number of post comments when determining which posts are the
most popular). That sproc is used to render the **Top Posts** section
on the Subtext dashboard page, and I *believe* the formula is based on
the following blog post:

{{< reference title="Rahien, Ayende (2007). Calculating most popular posts with SubText. 2007-03-09." linkHref="http://ayende.com/blog/2198/calculating-most-popular-posts-with-subtext" >}}

I chose not to use the existing sproc for a couple of reasons:

1. I'm not sure I like the idea of determining the popularity of blog posts
   based on the number of comments for each post. While this wouldn't necessarily
   "skew" the results significantly, it's more a matter of principle (especially
   considering the fact that some of my posts have tens of thousands of hits
   but the most comments I currently have on a single post is 15). The reality
   is that when you start getting blog posts with tens of thousands of hits
   (due to high rankings in Google search results), the influence of
   **AggCount** and number of comments becomes insignificant.
2. Since I had already created the **Most Recent Posts** user
   control using the Entity Framework and a simple LINQ query, I wanted to
   keep the implementation of the **Most Popular Posts** user
   control similar (to minimize development time and simplify maintenance of
   the code going forward).

Once I had the updated "plumbing" necessary to retrieve the list of most
popular posts from the Subtext database, I turned my attention to displaying
the results.

### PopularPosts.ascx

One of the primary goals in developing the Technology Toolbox site is ensuring
that semantic HTML is used to render the content. As such, I chose an ordered
list (`<ol>`)
as the basis for the **Most Popular Posts** section:

```
<div class="posts-most-popular">
    <h2>Most Popular Posts</h2>
    <ol>
        <li><a rel="bookmark" href="/blog/jjameson/archive/2007/06/17/issues-deploying-sharepoint-solution-packages.aspx">
            Issues Deploying SharePoint Solution Packages</a></li>
        <li><a rel="bookmark" href="/blog/jjameson/archive/2007/05/05/the-case-of-the-disappearing-hosts-file.aspx">
            The Case of the Disappearing Hosts File</a></li>
        <li><a rel="bookmark" href="/blog/jjameson/archive/2010/05/04/upgrade-team-foundation-server-2008-to-tfs-2010-and-sharepoint-server-2010.aspx">
            Upgrade Team Foundation Server 2008 to TFS 2010 (and SharePoint Server 2010)</a></li>
        <li>...</li>
        <li>...</li>
        ...
    </ol>
</div>
```

> **Note**
>
> Based on the appearance of the list items shown in Figure 1, you might think the **Most Popular Posts** section is rendered using an *uunordered* list (since there are no numbers next to the list items). However, that is simply the result of a custom "sprite" image (to render the arrows) and a little CSS. I'll cover that detail in a separate post. Rest assured, if you look at the page "naked" (i.e. with CSS disabled), you will see the list of popular posts with the corresponding rankings (i.e. 1-10).

As noted in my previous post, I typically define the semantic markup for
a feature using
[a static HTML prototype](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-3). Then I copy the sample HTML into a user control
and start replacing the sample content with ASP.NET controls to render the dynamic
content.

For the **Most Popular Posts** section on the site home page,
I added a new user control to the project (**PopularPosts.ascx**),
copied the sample HTML above into the user control, and then added an instance
of the control to the site home page.

In order to render an ordered list, I use an ASP.NET **Repeater** control, with a custom **HeaderTemplate**, **ItemTemplate**,
and **FooterTemplate**:

```
<div class="posts-most-popular">
    <h2>
        Most Popular Posts</h2>
    <asp:Repeater runat="server" ID="PostList">
        <HeaderTemplate>
            <ol>
        </HeaderTemplate>
        <ItemTemplate>
            <li><a href="<%# BlogHelper.GetEntryUrl(
                        (string) Eval("EntryName"),
                        (DateTime) Eval("DateSyndicated")) %>" rel="bookmark">
                <%# Eval("Title") %></a></li>
        </ItemTemplate>
        <FooterTemplate>
            </ol>
        </FooterTemplate>
    </asp:Repeater>
</div>
```

> **Note**
>
> Refer to my previous post if it isn't clear where the `BlogHelper.GetEntryUrl` method came from (or what it does).

In the corresponding code-behind for the user control, I added code to retrieve
the top 10 most popular posts and bind the results to the **Repeater** control:

```
using System;
using System.Linq;
using TechnologyToolbox.Caelum.Data;

namespace TechnologyToolbox.Caelum.Website.Controls
{
    public partial class PopularPosts : System.Web.UI.UserControl
    {
        protected void Page_Load(
            object sender,
            EventArgs e)
        {
            using (CaelumEntities context = new CaelumEntities())
            {
                var q = (from entry in context.Entries
                        join views in context.EntryViewCounts
                            on entry.ID equals views.EntryID
                        orderby (views.WebCount * 15)
                                + (views.AggCount * 10) descending
                        select entry).Take(10);

                PostList.DataSource = q;
                PostList.DataBind();
            }
        }
    }
}}
```

As my five-year-old daughter likes to say, "easy peasey, lemon squeezy!"

At this point, I ran a quick test to ensure the user control worked as expected.

With the feature "functionally" complete, I turned my attention to performance
tuning.

As with the control used to render the **Most Recent Posts**
ssection, I added a cache directive to the new user control (PopularPosts.ascx)
and specified a duration of one hour (3600 seconds):

```
<%@ OutputCache Duration="3600" VaryByParam="None" %>
```

This greatly reduces the number of roundtrips to SQL Server when users browse
to the home page of the site. Since I don't expect the list of popular posts
to change very often, I chose a much longer cache duration than RecentPosts.ascx.
(I could have chosen an even higher cache time -- such as 24 hours -- but, honestly,
I didn't see the need to go that high. Executing the above LINQ query once per
hour seems very reasonable, and I typically start with "gut feel" and then make
adjustments as necessary after running some load tests.)

Here is the complete source for PopularPosts.ascx:

```
<%@ Control Language="C#" AutoEventWireup="true" CodeBehind="PopularPosts.ascx.cs"
    Inherits="TechnologyToolbox.Caelum.Website.Controls.PopularPosts" %>
<%@ OutputCache Duration="3600" VaryByParam="None" %>
<%@ Import Namespace="TechnologyToolbox.Caelum.Website" %>
<div class="posts-most-popular">
    <h2>
        Most Popular Posts</h2>
    <asp:Repeater runat="server" ID="PostList">
        <HeaderTemplate>
            <ol>
        </HeaderTemplate>
        <ItemTemplate>
            <li><a href="<%# BlogHelper.GetEntryUrl(
                        (string) Eval("EntryName"),
                        (DateTime) Eval("DateSyndicated")) %>" rel="bookmark">
                <%# Eval("Title") %></a></li>
        </ItemTemplate>
        <FooterTemplate>
            </ol>
        </FooterTemplate>
    </asp:Repeater>
</div>
```

