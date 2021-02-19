---
title: "Integrating Bing Search with a Community Server Blog"
date: 2010-04-06T08:50:00-06:00
excerpt: "In one of yesterday's posts , I showed how you can easily filter the search results from Bing -- and other search engines -- to only show results from a specific site (e.g. my blog). 
 This morning it occurred to me that I could integrate this into my..."
aliases: ["/blog/jjameson/archive/2010/04/05/integrating-bing-search-with-a-community-server-blog.aspx", "/blog/jjameson/archive/2010/04/06/integrating-bing-search-with-a-community-server-blog.aspx"]
draft: true
categories: ["My System"]
tags: ["My System"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/06/integrating-bing-search-with-a-community-server-blog.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/06/integrating-bing-search-with-a-community-server-blog.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In [one of yesterday's posts](/blog/jjameson/2010/04/05/narrowing-search-results-to-a-specific-site-e-g-my-blog), I showed how you can easily filter the search results from Bing -- and other search engines -- to only show results from a specific site (e.g. my blog).

This morning it occurred to me that I could integrate this into my MSDN blog with relatively little effort.

Note that, as I've mentioned before, MSDN (and TechNet) blogs are currently powered by [Community Server](http://en.wikipedia.org/wiki/Community_Server). Consequently, the search options available to us are limited.

Yesterday I discovered that sometime last fall, the MSDN folks replaced the Community Server search functionality with Bing search. If you click the **Search** link on this page, you'll see what I mean. Unfortunately, when you use that search page, you get results from all MSDN blogs even though you may want to see the results from, say, just my blog.

> **Update (2009-04-20)**
>
> After [switching my blog from the **default** Community Server template](/blog/jjameson/2010/04/19/new-blog-template-and-styling) to the **Simple - right sidebar** template, I discovered that the MSDN team had already updated that template to replace the default "Search All Blogs" functionality with the "Search This Blog" functionality that one would expect (with a much nicer user experience, I might add). Consequently I removed my custom search box. Apparently, I should have abandoned the default CS template a long time ago ;-)

After digging around a little in the [Bing Developer Center](http://www.bing.com/developers), I found some [instructions for adding a basic search box to search just your website](http://help.live.com/help.aspx?project=WL_Webmasters&querytype=keyword&query=hcraescisab&mkt=en-us).

Here's the gist of it:

{{< blockquote "font-italic" >}}

To let your visitors search your website, add the following code to your page(s):

```
<!-- Site search from Bing-->
<form method="get" action="http://www.bing.com/search">
<input type="hidden" name="cp" value="CODE PAGE USED BY YOUR HTML PAGE" />
<input type="hidden" name="FORM" value="FREESS" />
  <table bgcolor="#FFFFFF">
    <tr>
      <td>
        <a href="http://www.bing.com/">
          <img src="http://www.bing.com/siteowner/s/siteowner/Logo_51x19_Dark.png"
            border="0" ALT="Bing" />
          </a>
      </td>
      <td>
        <input type="text" name="q" size="30" />
        <input type="submit" value="Search Site" />
        <input type="hidden" name="q1" value="site:YOUR DOMAIN NAME GOES HERE" />
      </td>
    </tr>
  </table>
</form>
<!-- Site Search from Bing -->
```

{{< /blockquote >}}

This seems pretty simple...but, unfortunately it's a little too simple (unless your site is serving up plain ol' HTML pages).

The problem is that pages served up by Community Server (or any ASP.NET application for that matter) already have a `<form>` element -- which usually spans the entire page. Consequently, if you simply copy/paste the HTML content above, you will end up with nested `<form>` elements. [While some browsers might be able to successfully handle nested `<form>`elements, it's definitely not valid HTML and therefore I don't recommend even attempting it.]

Instead, we need to substitute *equivalent* functionality without relying on a separate `<form>` element.

The good news is that the code snippet provided by the Bing folks above essentially defines a *contract* with the search service. [In other words, I'm assuming the Bing team isn't going to change the parameters that need to be specified, because that would break all of the sites that followed their simple instructions for adding a basic search box.]

Note that `method="get"` is specified on the `<form>` element. From this, we know that the values specified for any `<input>` elements are expected to be passed as query string parameters. With a little massaging of the HTML (to get rid of that nasty table-based layout) and crafting up some equivalent JavaScript, I came up with the following:

```
<script language="javascript" type="text/javascript">
    function searchKeywords_onKeyPress(e) {
        if (e == null) {
            e = window.event;
        }

        var key = e.keyCode ? e.keyCode : e.which;

        if (key == 13) {
            return false;
        }

        return true;
    }
    function showSearchResults() {
        var codePage = 1252;
        var siteUrl = "blogs.msdn.com/jjameson";

        var element = document.getElementById("searchKeywords");
        var keywords = element.value;

        if (keywords != null) {
            keywords = keywords.replace(/^\s+|\s+$/g, "");
        }

        if (keywords == null
        || keywords.length == 0) {
            alert("Please enter one or more words to search for.");
            return false;
        }

        var searchResultsUrl = "http://www.bing.com/search";

        searchResultsUrl += "?cp=" + codePage;
        searchResultsUrl += "&FORM=FREESS";
        searchResultsUrl += "&q=" + keywords;
        searchResultsUrl += " site:" + siteUrl;

        window.location.href = searchResultsUrl;
        return false;
    }
</script>
<div class="siteSearch">
    <div class="searchIntro">
        Search this blog using <a href="http://www.bing.com/">
        <img alt="Bing" src="http://www.bing.com/siteowner/s/siteowner/Logo_51x19_Dark.png"
          style="vertical-align: middle;" />
        </a></div>
    <input id="searchKeywords" size="24" type="text"
      onkeypress="return searchKeywords_onKeyPress(event);" />
    <input name="submitBingSearch" onclick="return showSearchResults();"
      type="button" value="Search" />
</div>
```

Note that a little more JavaScript is required than I would actually prefer in order to avoid handle a couple of interesting scenarios.

The first scenario is where a user presses the {{< kbd "Enter" >}} key instead of clicking the **Search** button. Depending on the browser, pressing the {{< kbd "Enter" >}} key in a form element may submit the form. However, since the Bing search feature is bypassing the form submission (and just redirecting directly to the search results page), we don't want to postback to the server. Otherwise, we would simply refresh the current page (and lose any search keywords specified by the user).

The second scenario is where someone doesn't specify any search keywords and instead just clicks the **Search** button. This is because if you don't specify any query terms (i.e. the "q" query string parameter), then you simply get redirected to [http://www.bing.com](http://www.bing.com/) (which would thoroughly confuse most people -- including myself).

If you have a Community Server blog, all you need to do is copy/paste the code provided above into the **News** field on the **Title, Description, and News** page, change the value of the **siteUrl** variable accordingly, and then click **Save**. Then you -- or anyone browsing your blog -- will be able to search your posts without having to specify a site filter each time.

> **Update**
>
> Right after I published this post, I discovered that Bing does not preserve the "q1" querystring parameter when submitting additional searches. In other words, if you search for "faceted search" from the search box on my blog, then everything works as expected (i.e. the results are limited to my blog posts); however if you subsequently change the search keywords on the Bing search results page (presumably because you want to search for something else on my blog), then the site filter (originally specified with the "q1" query string parameter) is lost and therefore you get results from the entire Web. Bummer.
>
> To resolve this issue, I modified the JavaScript to simply append the site filter to the "q" query string parameter instead.

