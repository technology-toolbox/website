---
title: "Web Standards Design with MOSS 2007, Part 1"
date: 2010-01-30T06:00:00-07:00
excerpt: "I've mentioned before that I became somewhat of a Web standards zealot several years ago. Consequently, regardless of whether I'm building Web sites using the core ASP.NET platform or Microsoft Office SharePoint Server (MOSS) 2007, I strive to ensure..."
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "WSS v3", "Web Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/01/30/web-standards-design-with-moss-2007-part-1.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/01/30/web-standards-design-with-moss-2007-part-1.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

I've mentioned before that I became somewhat of a [Web standards](http://en.wikipedia.org/wiki/Web_standards) zealot several years ago. Consequently, regardless of whether I'm building Web sites using the core ASP.NET platform or Microsoft Office SharePoint Server (MOSS) 2007, I strive to ensure that minimal, semantic HTML markup is used to render the site. I then leverage CSS rules to define the presentational aspects of the content. Note that this can sometimes seem like an epic battle between the Web developer and the underlying platform.

Anyone who has worked with SharePoint in sufficient depth knows all too well that the underlying markup generated by the out-of-the-box features can be very complex -- and definitely not "semantic." Table-based layouts are used virtually everywhere in MOSS 2007, and while CSS rules are utilized for much of the styling, you'll quickly encounter limitations when trying to customize the look-and-feel of your site.

However, it's important to understand that you *can* create Web sites using MOSS 2007 that are *mostly* Web standards compliant -- without abandoning a large amount of the "out-of-the-box goodness" that you get with MOSS 2007.

For example, I'm not willing to forgo using Web Parts entirely on a MOSS 2007 Web site just for the sake of achieving strict XHTML compliance. Others may disagree, but in my opinion, this isn't an acceptable tradeoff.

Some specific examples where OOTB Web Parts provide incredible value -- at the cost of some less-than-ideal markup -- are the search pages for the [Agilent Technolgies](http://www.chem.agilent.com/en-US/Search/Pages/default.aspx?k=purification) and [KPMG](http://www.kpmg.com/Global/en/Search/Pages/Results.aspx?k=corporate+governance) sites (both of which I originally built). Sure, you could choose to create your own search Web Parts that create the exact look-and-feel originally specified by your client, but that's not how a good SharePoint developer should approach the problem. As I've said numerous times before, I often find myself ripping out large amounts of customization and replacing the functionality with out-of-the-box features. This isn't always easy, but usually you can convince your client that changing the user experience a little (e.g. paging through search results) in order to eliminate large amounts of custom code is inevitably a very good thing.

If at this point you are thinking something along the lines of "but, Jeremy, that means all the Web sites built on SharePoint will essentially look the same," then I encourage you to compare the search result pages on the [Agilent](http://www.chem.agilent.com/en-US/Search/Pages/default.aspx?k=purification) and [KPMG](http://www.kpmg.com/Global/en/Search/Pages/Results.aspx?k=corporate+governance) sites. Yes there are clearly similarities between the two. For example, both sites provide a faceted search feature to allow users to quickly narrow their search results based on one or more property filters. However, neither one looks much like the out-of-the-box Search Center in MOSS 2007 (even though both of them are based on that site template).

On my most recent project, we are building an Internet-facing customer portal based on MOSS 2007. One of the first things I did shortly after joining the project was convert the existing proof-of-concept to use a Web standards design, stripping out large amounts of markup and simplifying the CSS dramatically.

Rather than defining all of the CSS layout from scratch, I chose to utilize the [960 Grid System](http://960.gs). For those of you that are still deeply rooted in using table-based layout, a grid is simply a combination of vertical columns, horizontal fields, and white space gutters (in other words, just say "no" to spacer GIFs and an ungodly number of nested tables).

The 960 Grid System provides two variants: 12 columns and 16 columns. However, we are currently only using the 12 column layout for this site. I should also note that we are using a fixed-width design (although there are "fluid" alternatives available based on the 960 Grid System). Bear in mind that things get considerably more complex when trying to support arbitrary sizing of the browser window (which is actually something that SharePoint does very well out-of-the-box with its table-based layout).

> **Tip**
>
> If you choose to use the 960 Grid System, I also highly recommend leveraging the [960 Gridder](http://gridder.andreehansson.se) as well. This combination makes the task of creating great looking Web pages much easier (even for someone, like me, who is much more of a *developer* than a *designer*).

To understand how the 960 Grid System is used on our SharePoint site, consider the following master page:

```
<%@ Master Language="C#" CodeBehind="Fabrikam.master.cs" Inherits="Fabrikam.Portal.Web.UI.FabrikamMasterPage, Fabrikam.Portal.Web,
        Version=1.0.0.0, Culture=neutral, PublicKeyToken=c8cdcbca6f69701f" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<%@ Import Namespace="Microsoft.SharePoint" %>
<%@ Register TagPrefix="SharePoint"
    Namespace="Microsoft.SharePoint.WebControls"
    Assembly="Microsoft.SharePoint, Version=12.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" %>
<%@ Register TagPrefix="WebPartPages"
    Namespace="Microsoft.SharePoint.WebPartPages"
    Assembly="Microsoft.SharePoint, Version=12.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" %>
<%@ Register TagPrefix="PublishingWebControls"
    Namespace="Microsoft.SharePoint.Publishing.WebControls"
    Assembly="Microsoft.SharePoint.Publishing, Version=12.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c" %>
<%@ Register TagPrefix="FabrikamWebControls"
    Namespace="Fabrikam.Portal.Web.UI.WebControls"
    Assembly="Fabrikam.Portal.Web, Version=1.0.0.0, Culture=neutral, PublicKeyToken=c8cdcbca6f69701f" %>
<%@ Register TagPrefix="PublishingConsole"
    TagName="Console"
    Src="~/_controltemplates/PublishingConsole.ascx" %>
<%@ Register TagPrefix="PublishingSiteAction"
    TagName="SiteActionMenu"
    Src="~/_controltemplates/PublishingActionMenu.ascx" %>
<%@ Register TagPrefix="Fabrikam"
    TagName="Welcome"
    Src="~/_controltemplates/Fabrikam/Welcome.ascx" %>
<html dir="<%$Resources:wss, multipages_direction_dir_value %>"
    runat="server" __expr-val-dir="ltr">
<head runat="server">
    <meta name="GENERATOR" content="Microsoft SharePoint" />
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <SharePoint:RobotsMetaTag runat="server" />
    <title id="onetidTitle"><asp:ContentPlaceHolder ID="PlaceHolderPageTitle" runat="server" /></title>
    <SharePoint:CssLink runat="server" />
    <!--Styles used for positioning, font and spacing definitions-->
    <SharePoint:CssRegistration
        runat="server"
        Name="<% $SPUrl:~sitecollection/Style Library/~language/Core Styles/controls.css %>" />
    <link rel="stylesheet" type="text/css"
        href="<% $SPUrl:~SiteCollection/Style Library/Fabrikam/Themes/Theme1/Fabrikam-Main.css%>" />
    <SharePoint:ScriptLink Name="init.js" runat="server" />
    <!--Placeholder for additional overrides-->
    <asp:ContentPlaceHolder ID="PlaceHolderAdditionalPageHead" runat="server" />
</head>
<body>
    <form runat="server" onsubmit="return _spFormOnSubmitWrapper();">
    <asp:ScriptManager runat="server" />
    <WebPartPages:SPWebPartManager ID="mainWPManager" runat="server" />
    <div class="container_12">
        <div id="masthead">
            <div id="logo">
                <a href="/" title="Go to Start Page">
                    <img src="/Style Library/Images/Fabrikam/fabrikam-logo.png" />
                </a>
            </div>
            <div id="welcomearea">
                <%--
                    Generally speaking, style attributes should be avoided in
                    HTML. Styling should instead be applied exclusively through
                    CSS.
                   
                    However the SharePoint SiteActionMenu control emits HTML
                    similar to the following:
                   
                        <table height="100%" class="ms-siteaction" ...>
                            ...
                        </table>
                   
                    When CSS is disabled, the height of the SiteActionMenu is
                    ridiculous (which makes it more difficult to troubleshoot
                    other styling issues). Consequently, limit the height of the
                    container <div> to avoid this issue.
                --%>
                <div id="siteactionmenu" style="height: 18px">
                    <PublishingSiteAction:SiteActionMenu runat="server" />
                </div>
                <div class="welcome">
                    <Fabrikam:Welcome id="WelcomeUserControl" runat="server" />
                </div>
            </div>
            <PublishingWebControls:AuthoringContainer runat="server">
                <PublishingConsole:Console runat="server" />
            </PublishingWebControls:AuthoringContainer>
        </div>
        <asp:ContentPlaceHolder ID="PlaceHolderMain" runat="server" />
    </div>
    <asp:Panel ID="Panel1" Visible="false" runat="server">
        <asp:ContentPlaceHolder ID="PlaceHolderSearchArea" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderTitleBreadcrumb" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderPageTitleInTitleArea" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderLeftNavBar" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderPageImage" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderBodyLeftBorder" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderNavSpacer" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderTitleLeftBorder" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderTitleAreaSeparator" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderMiniConsole" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderCalendarNavigator" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderLeftActions" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderPageDescription" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderBodyAreaClass" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderTitleAreaClass" runat="server" />
        <asp:ContentPlaceHolder ID="PlaceHolderBodyRightMargin" runat="server" />
    </asp:Panel>
    </form>
</body>
</html>
```

This is very similar to the [minimal master page example on MSDN](http://msdn.microsoft.com/en-us/library/aa660698.aspx) -- but with some important changes. I'll cover the important similarities as well as the differences in the remainder of this post.

Ignoring the differences in the page directives (which are irrelevant when discussing Web standards), the first thing you'll notice is that the master page specifies a DOCTYPE:

```
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
    "http://www.w3.org/TR/html4/loose.dtd">
```

You should always specify a DOCTYPE in your Web pages in order to avoid the dreaded [quirks mode](http://en.wikipedia.org/wiki/Quirks_mode) in Web browsers. Since, as I noted earlier, the markup emitted by SharePoint isn't always ideal, avoid the temptation to specify a strict DOCTYPE, such as:

```
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
   "http://www.w3.org/TR/html4/strict.dtd">
```

When I attended some training last summer on SharePoint 2010, I remember hearing that the new version is supposed to be compliant with XHTML strict. However, I don't know if that will actually hold true, but regardless, the current version -- as they say -- "is what it is."

In the MSDN minimal master page sample, the following items immediately follow the `<html>` element:

```
<WebPartPages:SPWebPartManager runat="server"/>
  <SharePoint:RobotsMetaTag runat="server"/>
```

Since the `<html>` element should only contain `<head>` and `<body>` elements, the [SPWebPartManager](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.webpartpages.spwebpartmanager.aspx) control should be placed within the `<form>` element inside the `<body>` element, and the [RobotsMetaTag](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.webcontrols.robotsmetatag.aspx) control should be placed within the `<head>` element (since it emits `<meta>` elements).

The next important difference between the sample MSDN master page and the one shown above is with regards to the links to CSS files. The MSDN sample only specifies the following:

```
<SharePoint:CssLink runat="server"/>
```

My master page, however, specifies the following:

```
<SharePoint:CssLink runat="server" />
    <!--Styles used for positioning, font and spacing definitions-->
    <SharePoint:CssRegistration
        runat="server"
        Name="<% $SPUrl:~sitecollection/Style Library/~language/Core Styles/controls.css %>" />
    <link rel="stylesheet" type="text/css"
        href="<% $SPUrl:~SiteCollection/Style Library/Fabrikam/Themes/Theme1/Fabrikam-Main.css%>" />
```

Note that the controls.css file is referenced in the out-of-the-box MOSS 2007 sample master pages (e.g. BlueBand.master) but for some reason was omitted from the MSDN sample page. I've seen some blog posts that say this CSS file should either be omitted entirely, or only rendered for content authors (by enclosing the declaration in an [AuthoringContainer](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.webcontrols.authoringcontainer.aspx) element). However, if you try to leverage as many OOTB SharePoint features as possible -- including the [SummaryLinkFieldControl](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.webcontrols.summarylinkfieldcontrol.aspx) for displaying lists of links -- then you should reference this CSS file (or be prepared to recreate numerous CSS rules that SharePoint would otherwise provide for you).

Also note that I use a regular HTML `<link>` element to link in my custom CSS file (Fabrikam-Main.css). This is because I want it to always appear last in the list of linked CSS files and, unfortunately, the SharePoint [CssLink](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.webcontrols.csslink.aspx) control has a nasty habit of rendering CSS links using its own ordering scheme -- rather than how you specify items with the [CssRegistration](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.webcontrols.cssregistration.aspx) control (hence why some of the sample CSS files included with SharePoint include the "zz1\_" prefix). Consequently I just use a regular HTML element to ensure I can override any CSS rules specified by SharePoint.

Before diving into the contents of my custom CSS file, let's first cover the purpose of my **Themes** and **Theme1** folders.

I have to admit that I initially liked the idea of [ASP.NET themes and skins](http://msdn.microsoft.com/en-us/library/ykzx33wh.aspx). However, that was before I became "enlightened" by the [CSS Zen Garden](http://www.csszengarden.com). Consequently, when I refer to a theme, I'm typically talking about a collection of CSS files that define the visual structure of site at a particular point in time. Since I want to provide the ability for a company to quickly revamp its Web site with a new look-and-feel, I utilize a **Themes** folder with individual subfolders that contain the CSS files (and related images) that define a specific look-and-feel.

While you can tell that I'm not really creative when it comes to names, you can easily imagine how we could add a **Theme2** folder -- along with a new set of CSS files, images, etc. -- and subsequently completely overhaul the visual appearance of a site -- simply by referencing the new Fabrikam-Main.css file in the **Theme2** folder. This is, after all, one of the compelling reasons for implementing a Web standards design.

The reason why the custom CSS file contains the "-Main" suffix is because it typically refers to other CSS files. For example, the first two lines of Fabrikam-Main.css are:

```
@import url('Fabrikam-Basic.css');
@import url('960.css');
```

At the risk of stating the obvious, the 960.css file is the one that comes with the 960 Grid System (without any changes whatsoever).

Depending on the need to support various browser versions, we might also need to include other CSS files (such as Fabrikam-IE.css or, Heaven forbid, Fabrikam-IE6.css). Fortunately, at least on this particular project, we have the privilege of playing the "unsupported" trump card if people complain that they don't like the way the site looks in Internet Explorer 6. Consequently, the number of CSS files is very small for this project.

Here are the contents of Fabrikam-Basic.css:

```
/* Reset styles to standardize formatting across various browsers (refer to
 * http://meyerweb.com/eric/tools/css/reset/ and
 * http://developer.yahoo.com/yui/reset/ for more info).
 *
 * Note that Eric's original approach is a little too "aggressive" for
 * SharePoint sites. For example, the "vertical-align: baseline;" reset rule
 * subsequently breaks many different areas of the site. Consequently, we base
 * our reset rules on the simpler set from the Yahoo! YUI team.
------------------------------------------------------------------------------*/
body, div, dl, dt, dd, ul, ol, li, h1, h2, h3, h4, h5, h6, pre, form, fieldset,
input, textarea, p, blockquote, th, td {
    margin: 0;
    padding: 0;
}
table {
    border-collapse: collapse;
    border-spacing: 0;
}
fieldset, img {
    border: 0;
}
address, caption, cite, code, dfn, em, strong, th, var {
    font-style: normal;
    font-weight: normal;
}
ol, ul {
    list-style: none;
}
caption, th {
    text-align: left;
}
h1, h2, h3, h4, h5, h6 {
    font-size: 100%;
    font-weight: normal;
}
q:before, q:after {
    content: '';
}
abbr, acronym {
    border: 0;
}
hr {
    border: 0;
    background: #000;
    color: black;
    height: 1px;
    margin: 0;
}
/* HTML elements (Ideally, this section should not contain any CSS classes,
 * but rather only contain generic CSS rules for HTML elements. These rules
 * essentially undo the "reset" rules above in order to achieve the desired
 * formatting across various "A-grade" browsers.
------------------------------------------------------------------------------*/
body {
    background: #fff;
    color: #58595b;
    font-family: Arial, Helvetica, Sans-Serif;
    font-size: small;
}
h1, h2, h3, h4, h5, h6, strong {
    font-weight: bold;
}
h1 {
    color: #868686;
    font-size: 2.3em;
    font-style: italic;
    font-weight: bold;
    margin: .67em 0;
}
h2 {
    color: #3a3b3c;
    font-size: 1.5em;
    font-weight: bold;
    margin: .83em 0;
}
h3 {
    color: #303435;
    font-size: 1.17em;
    font-weight: bold;
    margin: 1em 0;
}
h4 {
    font-weight: bold;
    margin: 1.33em 0;
}
h5 {
    font-size: 0.83em;
    font-weight: bold;
    margin: 1.67em 0;
}
h6 {
    font-size: 0.67em;
    font-weight: bold;
    margin: 2.33em 0;
}
abbr, acronym {
    border-bottom: 1px dotted #000;
    cursor: help;
}
blockquote {
    margin: 1em 40px;
}
em {
    font-style: italic;
}
dl, ol, ul {
    margin: 1em 1em 1em 2em;
}
dl dd {
    margin-left: 1em;
}
input[type="checkbox"] {
    margin-right: 5px;
}
input[type="submit"] {
    padding: 1px 4px;
}
ol li {
    list-style: decimal outside;
}
ul li {
    list-style: disc outside;
}
/* HACK: As noted above, this section should not contain rules for specific CSS
 * classes, but rather generic CSS rules for all HTML elements. However,
 * table-based layout is so rampant in MOSS 2007 that caution must be used when
 * applying rules to tables -- or else the page layout becomes atrocious.
 * Consequently, we use a CSS class to explicitly distinguish tables that are
 * intended to be shown to users ("display tables") from the generic tables used
 * for layout (i.e. "layout tables").
 */
th, table.displayTable td {
    border: 1px solid #000;
    padding: .5em;
}
th {
    font-weight: bold;
    text-align: center;
}
caption {
    margin-bottom: .5em;
    text-align: center;
}
fieldset {
  margin-left: 2px;
  margin-right: 2px;
  padding: 0.35em 0.625em 0.75em;
  border: 2px groove ThreeDFace;
}
/* HACK: See note above regarding use of "displayTable" class in this section */
p, fieldset, table.displayTable {
    margin-bottom: 1em;
}
hr {
    background: #fff;
    border-top: solid 1px #c3c3c3;
    margin: 1em 0;
}
```

The purpose of the "basic" CSS file -- as noted in the comments at the top of the file -- is to reset the default styling that frequently varies across different Web browsers and define the core styles of various HTML elements.

For the most part, these rules are based on the [YUI Reset CSS](http://developer.yahoo.com/yui/reset/). However, you can see that I had to make some minor changes in order to avoid fundamentally "breaking" SharePoint (due to the rampant table-based layout that I referred to earlier).

It's important to understand that changes to the Fabrikam-Basic.css file are not expected to occur very frequently. Rather, as new features are added to the SharePoint site, CSS rules are added or updated in Fabrikam-Main.css.

While most of the CSS rules in Fabrikam-Main.css wouldn't be of interest to most people (since they are specific to one particular Web site), it is worth highlighting a few of the rules:

```
/* =core (SharePoint core.css overrides)
------------------------------------------------------------------------------*/
.ms-pagebreadcrumb {
    border: 0;    
}
.ms-pagebreadcrumb a {
    background-color: inherit;
    color: inherit;
}
.ms-PartSpacingHorizontal {
    width: 0px !important;
}
.ms-MenuUIPopupBody table {
    color: #4C4C4C;
}
/* Override .ms-WPBody rules from core.css so that content within Web Parts
 * (e.g. a Content Editor Web Part) appears similar to other text on the page
 * (for example, as defined in the CSS rules for <body>) */
.ms-WPBody {
    font-family: inherit;
    font-size: inherit;
}
.ms-WPBody a:link, .ms-WPBody a:visited,
.ms-WPBody a:hover, .ms-WPBody a:active {
    color: #003399;
}
.ms-WPBody td {
    font-size: inherit;
    font-family: inherit;
}
.ms-WPBody span {
    font-size: inherit;
}
.ms-WPHeader td {
    border-bottom: 1px solid #303435;
}
.ms-WPTitle {
    color: #303435;
    font-family: inherit;
}
```

As noted in the comments above, these CSS rules are used to override rules specified in the out-of-the-box SharePoint CSS files.

Returning to the contents of the master page, the following element is used to encapsulate all of the page content:

```
<div class="container_12">
```

The `container_12` class refers to the 12-column template provided by the 960 Grid System.

Next comes the masthead, which contains the company logo, "welcome" area, and page editing toolbar (i.e. `<PublishingConsole:Console runat="server" />`). The main content for the page (i.e. `<asp:ContentPlaceHolder ID="PlaceHolderMain" runat="server" />`) comes after the masthead.

Finally, the master page includes a hidden panel that encapsulates all of the various content placeholders that are required by SharePoint, but should not be rendered on the page. As you can see, there are a number of placeholders in MOSS 2007 that are unfortunately used strictly for presentational markup (e.g. `PlaceHolderNavSpacer`and `PlaceHolderBodyLeftBorder`). When implementing a site based on Web standards, you should avoid these like the H1N1 virus. [For more information on the content placeholders specified in MOSS 2007, refer to my previous blog post ([MOSS 2007 Master Page Comparison](/blog/jjameson/2009/09/19/moss-2007-master-page-comparison)).]

This is getting to be a very long post, and there's still much I want to cover. Therefore, I'll tack "Part 1" onto the title, publish it, and get on with more fun things on a Saturday afternoon.

Don't fret, I'll cover many more details of Web standards design and SharePoint in the very near future eventually.

> **Update (2010-12-02)**
>
> Part 2 in this series in *finally* available ;-)
>
> **Web Standards Design with SharePoint, Part 2**
> [http://blogs.msdn.com/b/jjameson/archive/2010/12/02/web-standards-design-with-sharepoint-part-2.aspx](/blog/jjameson/2010/12/02/web-standards-design-with-sharepoint-part-2)

> **Update (2011-01-31)**
>
> Part 3 in this series provides a sample SharePoint master page and various page layouts based on the 960 Grid System.
>
> **Web Standards Design with SharePoint, Part 3**
> [http://blogs.msdn.com/b/jjameson/archive/2011/01/30/web-standards-design-with-sharepoint-part-3.aspx](/blog/jjameson/2011/01/30/web-standards-design-with-sharepoint-part-3)
