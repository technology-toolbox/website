---
title: "Analyzing My MSDN Blog"
date: 2009-11-02T00:42:00+08:00
excerpt: "According to my blog dashboard, this will be post #150 for my MSDN blog. So this morning I thought I would do something a little different by providing some analysis on my blog. 
 I extracted the data from Community Server into an Excel spreadsheet so..."
draft: true
categories: ["Personal"]
tags: ["Personal"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/11/02/analyzing-my-msdn-blog.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/11/02/analyzing-my-msdn-blog.aspx)
> 
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.


According to my blog dashboard, this will be post #150 for my MSDN blog. So this         morning I thought I would do something a little different by providing some analysis         on my blog.

I extracted the data from Community Server into an Excel spreadsheet so I could         easily create some pivot tables and charts. [I don't believe that the MSDN blogs         platform is running Harvest -- or if it is, I certainly don't have access to it.]

Here's a quick summary of my initial analysis:


<caption>            Table 1: MSDN Blog Usage Analysis</caption>|                     Metric<br>                 |                     Views<br>                 |                     AggViews<br>                 |                     Comments<br>                 |                     Combined Views<br>                 |
| --- | --- | --- | --- | --- |
|                     Total<br>                 |                     205,142<br>                 |                     139,377<br>                 |                     209<br>                 |                     344,519<br>                 |
|                     Maximum<br>                 |                     12,240<br>                 |                     2,855<br>                 |                     13<br>                 |                     14,445<br>                 |
|                     Minimum<br>                 |                     196<br>                 |                     306<br>                 |                     0<br>                 |                     678<br>                 |
|                     Average<br>                 |                     1,359<br>                 |                     923<br>                 |                     1.4<br>                 |                     2,282<br>                 |


Note that **Combined Views** is something I defined and is simply the         sum of **Views** and **AggViews**.

The following post provides a good explanation of the difference between "Views"         and "AggViews" in Community Server.

<cite>Eriksson, J-O (2006). Views statistics of your blog. 2006-07-24.</cite>
[http://weblogs.asp.net/joeriksson/archive/2006/07/24/Views-statistics-of-your-blog.aspx](http://weblogs.asp.net/joeriksson/archive/2006/07/24/Views-statistics-of-your-blog.aspx)


Here's the gist of it:


> Generally speaking, "Views" is the number of times somone viewed a post on the web             via a browser, and "AggViews" is the number of times someone viewed the post via             the RSS and Atom feeds.
> 
> More specifically, the web view count is only updated in the EntryView control.             This is displayed when you are viewing a single post. If you are viewing a list             of posts, such as on the blog home page, the view counts of the posts are not updated.


Based on the second paragraph, the numbers shown above are not entirely accurate         (specifically, they are less than the actual values) but my gut tells me they are         reasonably close. In other words, while some people may simply browse to [http://blogs.msdn.com/jjameson](http://blogs.msdn.com/jjameson) and start reading without ever clicking through         on an individual post, I don't think this is a large number.

After all, how many times do you find yourself truly "browsing" a site compared         with how many times you start from a search in order to quickly locate content of         interest? Then, of course, there's the case where someone starts reading a post         but then decides something like "hmmm...interesting...but not really what I was         looking for...time to move on..."

Note that I had to do a little tweaking for the minimum values computed in my spreadsheet         (in order to avoid showing all zeroes in the row) because I have a couple of unpublished         posts that obviously haven't been viewed by anyone except me. I'll talk more about         those in a moment.

I have to say that I was a little surprised by the total number of combined views.         While I'm sure 345,000 pales in comparison to the likes of [Scott Hanselman](http://www.hanselman.com/blog), [Scott Guthrie](http://weblogs.asp.net/scottgu/),         and [Joel Spolsky](http://joelonsoftware.com/), it's still a lot more         than I expected for my humble blog. Seeing the average number of RSS views for each         post exceed the 900 mark also makes me feel a little warm and fuzzy. So, for the         roughly 2,800 of you that have subscribed to my blog at one point or another, I         want to shout out a big "thank you!"

It's nice to know that the effort I put into writing blog posts are considered helpful         for a number of people. Anyway, moving on...

Here's a chart showing the combined views for each blog post that I've created so         far.

![Combined views for each blog post](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/15/r_MSDN%20blog%20-%20Combined%20Views.png)
            Figure 1: Combined views for each blog post

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/15/o_MSDN%20blog%20-%20Combined%20Views.png)


I didn't show the labels (i.e. blog post titles) on the X-axis for obvious reasons.         I find it interesting how some blog posts are really "hot" in comparison with the         average. Nothing mysterious here, this is obviously due to Internet search engines         such as Bing and Google. More on that in a moment.

Looking at the chart above, one of the first things I was interested in identifying         are my most popular posts, as well as my least popular posts.

Let's start with the top 10 posts:


<caption>            Table 2: Top 10 Blog Posts</caption>|                     Rank<br>                 |                     Post<br>                 |                     Combined Views<br>                 |
| --- | --- | --- |
|                     1<br>                 | [Issues Deploying SharePoint Solution Packages](/blog/jjameson/2007/06/17/issues-deploying-sharepoint-solution-packages) |                     14,445<br>                 |
|                     2<br>                 | [The Case of the Disappearing Hosts File](/blog/jjameson/2007/05/05/the-case-of-the-disappearing-hosts-file) |                     13,029<br>                 |
|                     3<br>                 | [Dumping MOSS 2007 Variations - Part 1](/blog/jjameson/2007/10/30/dumping-moss-2007-variations-part-1) |                     9,417<br>                 |
|                     4<br>                 | ["Error Creating Control" when using Microsoft Office SharePoint Designer 2007](/blog/jjameson/2007/03/22/error-creating-control-when-using-microsoft-office-sharepoint-designer-2007) |                     7,525<br>                 |
|                     5<br>                 | [Virtual Server<br>                        Issues and Recommendations for MOSS Virtual Environments](/blog/jjameson/2007/06/09/virtual-server-issues) |                     6,908<br>                 |
|                     6<br>                 | [Dumping MOSS 2007 Variations - Part 2](/blog/jjameson/2007/10/31/dumping-moss-2007-variations-part-2) |                     6,813<br>                 |
|                     7<br>                 | [Creating a Site Template in MOSS 2007 that Works in WSS v3](/blog/jjameson/2008/04/08/creating-a-site-template-in-moss-2007-that-works-in-wss-v3) |                     5,753<br>                 |
|                     8<br>                 | [Scope Dependencies for SharePoint Features](/blog/jjameson/2007/03/22/scope-dependencies-for-sharepoint-features) |                     5,607<br>                 |
|                     9<br>                 | [MOSS Development Environment and a Windows Update Bug](/blog/jjameson/2007/06/09/moss-development-environment-and-windows-update-bug) |                     5,307<br>                 |
|                     10<br>                 | [Installing<br>                        Visual Studio 2005 Service Pack 1](/blog/jjameson/2008/02/08/installing-visual-studio-2005-sp1) |                     5,199<br>                 |


Hmmm...nine of the top 10 posts are related to SharePoint. No big surprises there         ;-)

What about the bottom 10 posts?


<caption>            Table 3: Bottom 10 Blog Posts</caption>|                     Rank<br>                 |                     Post<br>                 |                     Combined Views<br>                 |
| --- | --- | --- |
|                     151<br>                 |                     Adventures in Upgrading TFS<br>                 |                     0<br>                 |
|                     150<br>                 |                     AutoEventWireup Issue in MOSS 2007<br>                 |                     0<br>                 |
|                     149<br>                 | [ArgumentNullException with Optional PublishingPage.Description Property  <br>                        (with some thoughts on breaking the build, too)](/blog/jjameson/2009/03/19/argumentnullexception-with-optional-publishingpage-description-property-with-some-thoughts-on-breaking-the-build-too) |                     678<br>                 |
|                     148<br>                 | [Eliminate MBSA Warnings Using Default Security Settings Policy](/blog/jjameson/2009/10/21/eliminate-mbsa-warnings-using-default-security-settings-policy) |                     679<br>                 |
|                     147<br>                 | [Constraining<br>                        Tables with CSS](/blog/jjameson/2009/11/01/constraining-tables-with-css) |                     728<br>                 |
|                     146<br>                 | [DataNavigateUrlFormatString Does Not Allow "javascript:"](/blog/jjameson/2009/10/26/datanavigateurlformatstring-does-not-allow-javascript) |                     773<br>                 |
|                     145<br>                 | [Add Rooler to Your Web Development Toolbox](/blog/jjameson/2009/10/21/add-rooler-to-your-web-development-toolbox) |                     826<br>                 |
|                     144<br>                 | [New MSDN<br>                        Theme on My Blog](/blog/jjameson/2009/10/21/new-msdn-theme-on-my-blog) |                     841<br>                 |
|                     143<br>                 | [KB 896861<br>                        and "Microsoft Fix it"](/blog/jjameson/2009/10/14/kb-896861-and-microsoft-fix-it) |                     849<br>                 |
|                     142<br>                 | [Latest Version of Opera Ignores Hosts File](/blog/jjameson/2009/10/29/latest-version-of-opera-ignores-hosts-file) |                     850<br>                 |


I suppose that I really should have thrown out the first two items because these         posts are not published.

The **Adventures in Upgrading TFS** post was something I started back         in March 2008 when I ran into a couple of snags upgrading the instance of Team Foundation         Server 2005 in the ["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) to TFS 2008. I thought I would eventually get around         to finishing that post, but apparently not. Perhaps some other morning...however,         I'm not sure it would help very many people after all this time. I certainly hope         most organizations running TFS have upgraded by now.

The **AutoEventWireUp Issue in MOSS 2007** post is something I really         should finish. I started it back in June 2008 but somehow it fell off my plate.         Stay tuned...I'll try to get to that this week. No, I won't "try" -- I will get         to that this week. I promise.

In the meantime, back to this post.

It's also not suprising to see that many of the items in the bottom 10 are recent         posts. As such, one would expect them to have fewer hits than others.

Since it's just so incredibly easy to do in Excel, I decided to see how my authoring         of blog posts has varied over time.

Let's start with the total number of posts by year:

![Blog posts created by year](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/15/r_MSDN%20blog%20-%20Total%20Posts%20by%20Year.png)
            Figure 2: Blog posts created by year

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/15/o_MSDN%20blog%20-%20Total%20Posts%20by%20Year.png)


Hmmm...apparently 2008 wasn't a very good year for me from a blogging perspective.         Fortunately, the number for 2009 is much better -- and we still have almost two         months to go!

Let's look at the total number of posts by month:

![Blog posts created by month](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/15/r_MSDN%20blog%20-%20Total%20Posts%20by%20Month.png)
            Figure 3: Blog posts created by month

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/15/o_MSDN%20blog%20-%20Total%20Posts%20by%20Month.png)


Well, that certainly is, um...*erratic*. Oh, and where is December?!

Ouch...it looks like for the last two years I have completely neglected my blog         in that month.

Will this year be any different? Hmmm...wait and see ;-)

I can tell you that I'll be taking quite a bit of vacation again this year in December         and my wife won't appreciate it very much if I blog when I should be working on         refinishing our master bathroom. However, I'll try not to skip that month entirely         once again.

I read one of Scott Hanselman's posts a while ago that talked about calculating         your PPM (Posts Per Month) -- or was it BPM? I can't remember. Anyway, it seems         like I could definitely be a little more consistent throughout the year. Trying         to maintain a steady rhythm when it comes to blogging is always a challenge when         you consider how much else there is to do on any given day.

The last area that I want to cover in this post is with regards to Internet search         results -- which I briefly mentioned earlier. In other words, while it is interesting         that blog posts like [Issues Deploying SharePoint Solution Packages](/blog/jjameson/2007/06/17/issues-deploying-sharepoint-solution-packages) and [The Case of the Disappearing Hosts File](/blog/jjameson/2007/05/05/the-case-of-the-disappearing-hosts-file) are at the top of the list,         what I find more interesting is *why* they appear at the top of the list.

Let's start with the most popular post. Drilling down on the 12,246 views in the         Community Server dashboard (which does not include "AggViews") to see the list of         referrals, I encountered the largest HTML table I've ever seen on a page (over 5,500         rows). After a couple of attempts, I managed to copy all of the data into my Excel         workbook and then sort by **Hits **descending.
<!-- HACK: Community Server replaces the URLs, which blows out the table  -->

<caption>            Table 4: Top referrals by URL for post - <a href="/blog/jjameson/2007/06/17/issues-deploying-sharepoint-solution-packages">
                Issues Deploying SharePoint Solution Packages</a></caption>|                     URL<br>                 |                     Hits<br>                 |                     Last Date<br>                 |
| --- | --- | --- |
| [http://decatec.it/blogs/2007/06/18/sharepoint+deployment+tecniques.aspx](http://decatec.it/blogs/2007/06/18/sharepoint+deployment+tecniques.aspx) |                     83<br>                 |                     Oct 12 2009, 01:58 AM<br>                 |
| [http://google.com/search?q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications.&ie=utf-8&oe=utf-8&aq=t&rls=org.mozilla:en-us:official&client=firefox-a](http://google.com/search?q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications.&amp;ie=utf-8&amp;oe=utf-8&amp;aq=t&amp;rls=org.mozilla:en-us:official&amp;client=firefox-a) |                     76<br>                 |                     Oct 12 2009, 10:02 AM<br>                 |
| [http://stevepietrek.com/2007/06/17/links-6172007/](http://stevepietrek.com/2007/06/17/links-6172007/) |                     69<br>                 |                     Oct 28 2009, 02:41 AM<br>                 |
| [http://google.com/search?q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications&ie=utf-8&oe=utf-8&aq=t&rls=org.mozilla:en-us:official&client=firefox-a](http://google.com/search?q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications&amp;ie=utf-8&amp;oe=utf-8&amp;aq=t&amp;rls=org.mozilla:en-us:official&amp;client=firefox-a) |                     45<br>                 |                     May 04 2009, 03:04 PM<br>                 |
| [http://google.com/search?q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications.&rls=com.microsoft:en-us&ie=utf-8&oe=utf-8&startindex=&startpage=1](http://google.com/search?q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications.&amp;rls=com.microsoft:en-us&amp;ie=utf-8&amp;oe=utf-8&amp;startindex=&amp;startpage=1) |                     44<br>                 |                     Oct 28 2009, 02:03 PM<br>                 |
| [http://google.com/search?q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deploy&ie=utf-8&oe=utf-8&aq=t&rls=org.mozilla:en-us:official&client=firefox-a](http://google.com/search?q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deploy&amp;ie=utf-8&amp;oe=utf-8&amp;aq=t&amp;rls=org.mozilla:en-us:official&amp;client=firefox-a) |                     42<br>                 |                     Oct 30 2009, 03:18 PM<br>                 |
| [http://google.com/search?hl=en&q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications](http://google.com/search?hl=en&amp;q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications) |                     37<br>                 |                     Feb 04 2009, 04:07 PM<br>                 |
| [http://google.co.uk/search?hl=en&q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications.&meta=](http://google.co.uk/search?hl=en&amp;q=this+solution+contains+resources+scoped+for+a+web+application+and+must+be+deployed+to+one+or+more+web+applications.&amp;meta=) |                     35<br>                 |                     Oct 27 2009, 08:10 AM<br>                 |
| [http://google.com/search?hl=en&q=this+solution+contains+no+resources+scoped+for+a+web+application+and+cannot+be+deployed+to+a+particular+web+application](http://google.com/search?hl=en&amp;q=this+solution+contains+no+resources+scoped+for+a+web+application+and+cannot+be+deployed+to+a+particular+web+application) |                     34<br>                 |                     Oct 27 2009, 11:56 PM<br>                 |
| [http://google.com/search?q=this+solution+contains+no+resources+scoped+for+a+web+application+and+cannot+be+deployed+to+a+particular+web+application&ie=utf-8&oe=utf-8&aq=t&rls=org.mozilla:en-us:official&client=firefox-a](http://google.com/search?q=this+solution+contains+no+resources+scoped+for+a+web+application+and+cannot+be+deployed+to+a+particular+web+application&amp;ie=utf-8&amp;oe=utf-8&amp;aq=t&amp;rls=org.mozilla:en-us:official&amp;client=firefox-a) |                     33<br>                 |                     Nov 01 2009, 06:57 PM<br>                 |
| [http://social.technet.microsoft.com/forums/en-us/sharepointadmin/thread/0881326b-7cb6-4198-9bac-7df6eaed9dde](http://social.technet.microsoft.com/forums/en-us/sharepointadmin/thread/0881326b-7cb6-4198-9bac-7df6eaed9dde) |                     32<br>                 |                     Oct 17 2009, 03:41 PM<br>                 |


However, adding up the numbers for the top 10 referrals accounts for less than 5%         of the 12,246 views for this post. Upon further inspection, I found that only 8,400         of the 12,200 views actually have referral data.

Breaking down the referrals by domain name, I found the following:


<caption>            Table 5: Top referrals by domain for post - <a href="/blog/jjameson/2007/06/17/issues-deploying-sharepoint-solution-packages">
                Issues Deploying SharePoint Solution Packages</a></caption>|                     Domain<br>                 |                     Hits<br>                 |
| --- | --- |
|                     google.com<br>                 |                     3,893<br>                 |
|                     google.co.uk<br>                 |                     628<br>                 |
|                     google.co.in<br>                 |                     519<br>                 |
|                     google.com.au<br>                 |                     310<br>                 |
|                     google.ca<br>                 |                     288<br>                 |
|                     google.nl<br>                 |                     211<br>                 |
|                     google.de<br>                 |                     198<br>                 |
|                     search.live.com<br>                 |                     129<br>                 |
|                     google.fr<br>                 |                     125<br>                 |
|                     decatec.it<br>                 |                     99<br>                 |


Note that the post was created back in June 2007, which helps explain why you see         **search.live.com** instead of **bing.com** in the top         10 list.

To help me get a better feel for the Google vs. Microsoft ratio, I grouped the results         into categories and subcategories.

![Referrals by category for post - Issues Deploying SharePoint Solution Packages](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/15/r_MSDN%20blog%20-%20Referrals%20by%20Category.png)
            Figure 4: Referrals by category for post - [Issues Deploying SharePoint Solution Packages](/blog/jjameson/2007/06/17/issues-deploying-sharepoint-solution-packages)

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/15/o_MSDN%20blog%20-%20Referrals%20by%20Category.png)


It turns out that the subcategories were really only interesting for the purposes         of identifying the top-level categories.

What's up with Alta Vista and AOL? Does anyone still use those search engines? Evidently         a few people in the last couple of years ;-)

I have a strong suspicion that if I went through the same exercise for my second         most popular post, I would see very similar results.

I'm not sure how useful this information might be to others, but I found it very         enlightening. It gives me some great things to keep in mind when I sit down to work         on my blog.

