---
title: "Implementing Google Site Search (a.k.a. Building TechnologyToolbox.com, part 21)"
date: 2012-02-02T05:45:10-07:00
excerpt: "In this post, I provide another step-by-step walkthrough of a feature, this time sharing the details of how I implemented the search functionality on TechnologyToolbox.com."
aliases: ["/blog/jjameson/archive/2012/02/01/building-technologytoolbox-com-part-21.aspx", "/blog/jjameson/archive/2012/02/02/building-technologytoolbox-com-part-21.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["Web Development"]
---

I clearly remember the sunny day back in September sitting on a boat at Chatfield
Lake discussing the new Technology Toolbox website with my good friend Scott.
He was pestering me (all in good faith, of course) about when I was going to
send out an announcement about my new company.

I told him that I had a few more features I needed to finish on the website
before considering it complete -- one of them being Site Search. Scott, being
the occasional jester that he is, mocked my reply and said I shouldn't bother
with "fluff" like that.

All joking aside, if you have a website today that does not provide a search
box somewhere on the home page (preferably "above the fold" if not at the very
top), then from a usability perspective, I would probably say your site stinks.
Sorry, but there really is *no excuse* for that anymore -- no matter
how small your site is. [Okay, okay, I suppose if you have a site with less
than three pages, then I should let it slide.]

We may call it *browsing*, but people very rarely browse for information
anymore. Ever since the days of Alta Vista (and probably before that, but that
is as far back as I can recall) the human race has become more and more dependent
on searching.

During my time at Microsoft, I worked on the Site Search features for a couple
of fairly large websites (most notably, [Agilent
LSCA](http://chem.agilent.com) and [KPMG](http://www.kpmg.com)) using MOSS 2007 and SharePoint
2010. While I certainly would have liked to have used SharePoint to power the
search features for TechnologyToolbox.com, it simply wasn't practical. You need
a little more "critical mass" (for example, dedicated servers rather than shared
hosting) before SharePoint Search is a viable option.

I chose instead to use [Google
Site Search](http://www.google.com/sitesearch/). Here is how I implemented it...

### Step 1: Set up your search engine and write a small check to Google

Depending on the nature of your website and your target audience, you might
be able to get by with Google's free version. On the Technology Toolbox website,
we really don't want to display advertisements on the search results page. In
lieu of ads, Google requires compensation in another form. That seems reasonable
to me.

Once you've provided Google with a few parameters (such as the domain names
to include) and paid the annual subscription fee, they provide you with a unique
key (to limit search results to "your search engine") as well as sample HTML,
JavaScript, and CSS to integrate with your website.

### Step 2: Create the search results page

Next, I added a new page (Search.aspx) to the
[static HTML prototype](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-3) for TechnologyToolbox.com. Then I copied the HTML,
JavaScript, and CSS provided by Google into the page.

At that point, the search page looked like Figure 1.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Google-Site-Search-1-296x600.png"
alt="Google Site Search (with borders)"
class="screenshot"
height="600"
width="296"
title="Figure 1: Google Site Search (with borders)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Google-Site-Search-1-989x2006.png)

The hideous black borders are a result of the fact that Google Site Search
uses table-based layout all over the place (and I just happened to have defined
a CSS rule to show borders on tables). Yuck.

Well, as I'm fond of saying, "It is what it is."

To get rid of the borders and extraneous spacing, I added a few CSS rules
to override the table styles on the Search page:

```
#search table th,
#search table td {
    border: none;
    padding: inherit;
}
```

As you can see in the following screenshot, this improves things considerably.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Google-Site-Search-2-328x600.png"
alt="Google Site Search (without borders)"
class="screenshot"
height="600"
width="328"
title="Figure 2: Google Site Search (without borders)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Google-Site-Search-2-989x1810.png)

However, the search results still have the "look-and-feel" of Google -- not
Technology Toolbox. I began resolving the styling issues by removing the link
to Google's CSS file and adding the following rules to a CSS file for this website:

```
/* =search (Search page)
------------------------------------------------------------------------------*/
/* HACK: Google uses table-based layout for search results (Yuck!) */
#search table {
margin-bottom: 0px;
}
#search table th,
#search table td {
border: none;
padding: inherit;
}
#search .gsc-result-info {
border-bottom: 1px solid #c8d7eb;
color: #959595;
padding-bottom: 5px;
padding-left: 10px;
}
#search .gsc-control-cse {
font-family: inherit;
}
#search input.gsc-input {
border-color: #c8d7eb;
padding: 3px 0 3px 3px;
}
/*
#search input.gsc-search-button {
border-color: #666666;
background-color: #CECECE;
}
#search .gsc-tabHeader.gsc-tabhInactive {
border-color: #E9E9E9;
background-color: #E9E9E9;
}
#search .gsc-tabHeader.gsc-tabhActive {
border-top-color: #FF9900;
border-left-color: #E9E9E9;
border-right-color: #E9E9E9;
background-color: #fff;
}
#search .gsc-tabsArea {
border-color: #E9E9E9;
}
#search .gsc-webResult.gsc-result,
#search .gsc-results .gsc-imageResult {
border-color: #fff;
background-color: #fff;
}
#search .gsc-webResult.gsc-result:hover,
#search .gsc-imageResult:hover {
border-color: #fff;
background-color: #fff;
}
*/
#search .gs-webResult.gs-result a.gs-title:link,
#search .gs-webResult.gs-result a.gs-title:link b,
#search .gs-imageResult a.gs-title:link,
#search .gs-imageResult a.gs-title:link b {
color: #3c78c3;
}
#search .gs-webResult.gs-result a.gs-title:visited,
#search .gs-webResult.gs-result a.gs-title:visited b,
#search .gs-imageResult a.gs-title:visited,
#search .gs-imageResult a.gs-title:visited b {
color: #1e4173;
}
#search .gs-webResult.gs-result a.gs-title:hover,
#search .gs-webResult.gs-result a.gs-title:hover b,
#search .gs-imageResult a.gs-title:hover,
#search .gs-imageResult a.gs-title:hover b {
color: #3c78c3;
}
#search .gs-webResult.gs-result a.gs-title:active,
#search .gs-webResult.gs-result a.gs-title:active b,
#search .gs-imageResult a.gs-title:active,
#search .gs-imageResult a.gs-title:active b {
color: #3c78c3;
}
/*
#search .gsc-cursor-page {
color: #959595;
}
#search a.gsc-trailing-more-results:link {
color: #959595;
}
#search .gs-webResult .gs-snippet,
#search .gs-imageResult .gs-snippet {
color: #000000;
}
#search .gs-webResult div.gs-visibleUrl,
#search .gs-imageResult div.gs-visibleUrl {
color: #008000;
}
*/
#search .gs-webResult div.gs-visibleUrl-short {
display: none;
}
#search .gs-webResult div.gs-visibleUrl-long {
display: block;
}
#search .gsc-cursor-box {
margin-top: 10px;
}
#search .gsc-results .gsc-cursor-box .gsc-cursor-page {
background-color: #3c78c3;
color: #fff;
padding: 3px 6px;
text-decoration: none;
}
#search .gsc-results .gsc-cursor-box .gsc-cursor-current-page {
background-color: #1e4173;
color: #fff;
padding: 3px 6px;
}
```

> **Note**
>
> I commented out a number of CSS rules (copied from Google's CSS file)
> because I could not find instances in the HTML returned by Google where
> these are used. This may depend on which features of Google Site Search
> you choose to implement.

Figure 3 shows the corresponding results.

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Google-Site-Search-3-381x600.png"
alt="Google Site Search (final)"
class="screenshot"
height="600"
width="381"
title="Figure 3: Google Site Search (final)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/Google-Site-Search-3-989x1556.png)

At this point, the styling of search results appeared consistent with the
rest of the website. However, I later discovered scenarios not covered by the
CSS rules I had copied from Google -- specifically when an error occurs or no
search results are found. To provide consistent styling in those scenarios,
I added a few more CSS rules:

```
#search .gs-error-result .gs-snippet {
    background-color: #fbeded;
    border: 1px solid #bd1c1c;
}
#search .gs-no-results-result .gs-snippet {
    background-color: #eee;
    border: 1px solid #c8d7eb;
}
```

### Step 3: Create custom search box

While Google provides sample HTML and JavaScript for a search box, I wanted
something a little more "polished" for TechnologyToolbox.com.

I started out by adding the following HTML to the master page in the static
HTML prototype:

```
  <div id="siteSearch">
    <h2>
      Search</h2>
    <input type="text" id="searchKeywords" /><input type="image" alt="Search"
      src="Images/icon-search-22x22.png" />
  </div>
```

...and then adding some CSS rules to position the search box at the desired
location in the masthead.

Next, I added a little JavaScript to redirect to the search results page
(Search.aspx) when the search icon is clicked:

```
  <div id="siteSearch">
    <h2>
      Search</h2>
    <input type="text" id="searchKeywords" />
    <a href="javascript:submitSearch()">
      <img alt="Search" src="Images/icon-search-22x22.png" /></a>
    <script type="text/javascript">
      function submitSearch() {
        var $searchKeywords = $('#searchKeywords');

        var keywords = $searchKeywords.val();

        window.location.href = "/Search.aspx?q=" + encodeURIComponent(keywords);
      }
    </script>
  </div>
```

At this point, I copied the code from the prototype (created in Expression
Web) to the actual ASP.NET Web application (in Visual Studio).

### Step 4: Improve the user experience

While the basic functionality worked as expected at this point, there were
a few areas for improving the user experience.

First, rather than having to click the search icon, users should be able
to simply press {{< kbd "Enter" >}} after typing one or more search terms. [This
obviously needs to work in all the major browsers, so I obviously didn't want
to mimic the
[functionality in MOSS 2007](/blog/jjameson/2009/10/01/enter-key-does-not-submit-search-in-moss-2007-from-firefox).]

Second, to help draw attention to the search box, it should contain "Search..."
by default, but when somebody clicks in the box the default text should be removed.

These two behaviors are easy to implement using jQuery:

```
    <div id="siteSearch">
      <h2>
        Search</h2>
      <input type="text" id="searchKeywords" />
      <a href="javascript:submitSearch($('#searchKeywords'))"><img alt="Search"
        src="/Images/icon-search-22x22.png" /></a>
      <script type="text/javascript">
        $(document).ready(function () { configureSearchBox($('#searchKeywords')); });
      </script>
    </div>
```

Here is the **configureSearchBox** function:

```
function configureSearchBox(searchBox)
{
    searchBox.val("Search...");

    searchBox.focus(function ()
    {
        if (this.value === "Search...")
        {
            this.value = "";
        }
    })
    .blur(function ()
    {
        if (this.value === "")
        {
            this.value = "Search...";
        }
    })
    .keypress(function (event)
    {
        if (event.which == 13)
        {
            event.preventDefault();
            submitSearch($(this));
        }
    });
}
```

Third, the search feature should prevent users from submitting a search when
no keywords are specified (i.e. the user presses {{< kbd "Enter" >}} without typing
one or more terms) and when the search box contains the default text (i.e. the
user clicks the search icon without first typing one or more terms). This is
similar to the out-of-the-box behavior of SharePoint Search.

This requires a little more code in the **submitSearch** function
shown earlier:

```
function submitSearch(searchBox)
{
    var keywords = searchBox.val();

    if (keywords == "" || keywords == "Search...")
    {
        alert("Please enter one or more search keywords.");
        searchBox.focus();
        return;
    }

    window.location.href = "/Search.aspx?q=" + encodeURIComponent(keywords);
}
```

As I mentioned before, I added the search box to the top of the master page.
Consequently it appears at the top of every page -- including the search results
page (Search.aspx). However on that page, it doesn't make sense to show two
search boxes (in other words, the one rendered by the master page and another
included in the HTML that is inserted into the page by Google).

To hide the search box in the masthead, I added the following to Search.aspx:

```
<asp:Content runat="server" ContentPlaceHolderID="AdditionalHeadContent">
<style type="text/css">
  #siteSearch {
    visibility: hidden;
  }
</style>
</asp:Content>
```

### Step 5: Track search terms using Google Analytics

Technology Toolbox uses Google Analytics as well as Google Site Search. To
support Site Search reports in Google Analytics, you need to specify your analytics
key when executing the search.

After some refactoring, I ended up with an ASP.NET control that encapsulates
the scripts used for Google Site Search and also contains logic to conditionally
specify the analytics key.

> **Update (2011-02-20)**
>
> The code below originally specified "http://www.google.com/jsapi" which
> resulted in mixed mode content when browsing the site using HTTPS. I
> updated this to use a
> [protocol-relative URL](/blog/jjameson/2012/02/20/use-protocol-relative-urls-to-avoid-mixed-mode-content) instead ("//www.google.com/jsapi").

```
using System;
using System.Web.UI;
using System.Web.UI.WebControls;

namespace TechnologyToolbox.Caelum.Website.Controls
{
    [ToolboxData("<{0}:SearchResults runat=server></{0}:SearchResults>")]
    public class SearchResults : Control
    {
        private const string searchEngineUniqueId =
            "005042176703738775616:sqv29_g_cee";

        protected override void Render(
            HtmlTextWriter writer)
        {
            if (writer == null)
            {
                throw new ArgumentNullException("writer");
            }

            bool enableAnalytics = AnalyticsHelper.IsAnalyticsEnabled(
                this.Page.Request);

            writer.WriteLineNoTabs(
                "<div id='cse' style='width: 100%;'>Loading</div>");

            writer.WriteLineNoTabs(
                "<script src='//www.google.com/jsapi'"
                    + " type='text/javascript'></script>");

            writer.WriteBeginTag("script");
            writer.WriteAttribute("type", "text/javascript", false);
            writer.Write(HtmlTextWriter.TagRightChar);

            writer.WriteLineNoTabs(
@"
    function parseQueryFromUrl() {
        var queryParamName = 'q';
        var search = window.location.search.substr(1);
        var parts = search.split('&');
        for (var i = 0; i < parts.length; i++) {
            var keyvaluepair = parts[i].split('=');
            if (decodeURIComponent(keyvaluepair[0]) == queryParamName) {
                return decodeURIComponent(keyvaluepair[1].replace(/\+/g, ' '));
            }
        }
        return '';
    }");

            if (enableAnalytics == true)
            {
                string analyticsKey = AnalyticsHelper.GetAnalyticsKey(
                    this.Page.Request);

                writer.WriteLineNoTabs(
@"
    var _gaq = _gaq || [];
    _gaq.push(['_setAccount', '" + analyticsKey + @"']);
    function _trackQuery(control, searcher, query) {
        var loc = document.location;
        var url = [
            loc.pathname,
            loc.search,
            loc.search ? '&' : '?',
            encodeURIComponent('q'),
            '=',
            encodeURIComponent(query)
        ];
        _gaq.push(['_trackPageview', url.join('')]);
    }");
            }

            writer.WriteLineNoTabs(
@"
    google.load('search', '1', { language: 'en' });
    google.setOnLoadCallback(function () {
        var customSearchControl = new google.search.CustomSearchControl('"
+ searchEngineUniqueId + @"');
        customSearchControl.setResultSetSize(google.search.Search.FILTERED_CSE_RESULTSET);");

            if (enableAnalytics == true)
            {
                writer.WriteLineNoTabs(
@"        customSearchControl.setSearchStartingCallback(null, _trackQuery);");
            }

            writer.WriteLineNoTabs(
@"        customSearchControl.draw('cse');
        var queryFromUrl = parseQueryFromUrl();
        if (queryFromUrl) {
            customSearchControl.execute(queryFromUrl);
        }
    }, true);");

            writer.WriteEndTag("script");
        }
    }
}
```

Here is the complete source for Search.aspx:

```
<%@ Page Title="Search - Technology Toolbox" Language="C#"
  MasterPageFile="~/Default.master" %>

<%@ Register Src="~/Controls/Breadcrumb.ascx" TagName="Breadcrumb"
  TagPrefix="uc1" %>
<%@ Register TagPrefix="caelum"
  Namespace="TechnologyToolbox.Caelum.Website.Controls"
  Assembly="TechnologyToolbox.Caelum.Website, Version=1.0.0.0,
    Culture=neutral, PublicKeyToken=f55b5d7768fcda39" %>
<asp:Content runat="server" ContentPlaceHolderID="AdditionalHeadContent">
  <style type="text/css">
    #siteSearch
    {
      visibility: hidden;
    }
  </style>
</asp:Content>
<asp:Content ContentPlaceHolderID="MainContent" runat="server">
  <div id="search">
    <div id="content" class="container_12">
      <div id="pageHeader">
        <h1>
          Search</h1>
        <uc1:Breadcrumb runat="server" />
      </div>
      <div id="contentMain" class="grid_12">
        <caelum:SearchResults runat="server" />
      </div>
    </div>
  </div>
</asp:Content>
```

I will describe the process of implementing Google Analytics in
[my next post](/blog/jjameson/2012/02/03/building-technologytoolbox-com-part-22).

