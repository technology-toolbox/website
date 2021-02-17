---
title: "Creating a style guide and color palette for a Web application (a.k.a. Building TechnologyToolbox.com, part 4)"
date: 2011-11-03T07:54:12-07:00
excerpt: "In my previous post, I described how I typically create a \"static HTML prototype\" 
for an ASP.NET or SharePoint Web application. By working directly in HTML at the 
beginning, I can rapidly define the structure of the content and subsequently create the corresponding CSS rules to style the pages..."
draft: true
categories: ["My System", "Development"]
tags: ["SharePoint 
		2010", "Web Development"]
---

In [my previous post](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-3), I described how I typically create a "static HTML prototype"  for an ASP.NET or SharePoint Web application. By working directly in HTML at the  beginning (rather than starting with ASP.NET controls or Web Parts), I can rapidly  define the structure of the content (using semantic HTML) and subsequently create  the corresponding CSS rules to style the pages.

As part of the prototype, I like to include a "Style Guide" page that contains  a variety of HTML elements -- including the specific `id` and `class` attributes -- that will eventually  be used in the live website.

I like to think of the "Style Guide" as a form of TDD (Test Driven Development)  for the cascading style sheet, because you start out with some plain looking content  (i.e. semantic HTML) and then add CSS rules to get it to render as expected in the  browser. Then when someone encounters a UI bug (in other words, when something doesn't  appear quite right) you subsequently tweak the Style Guide to reproduce the issue  and then update the CSS to resolve the bug.

### Style-Guide.aspx

Here is the current content of the Style Guide for the static HTML prototype  for TechnologyToolbox.com (i.e. $/Caelum/Dev/CaelumPrototype/Style-Guide.aspx):

```
<%@ Page language="C#" %>

<asp:Content runat="server" ContentPlaceHolderID="AdditionalHeadContent">
  <style type="text/css">
  .color-swatch {
    float: left;
    min-height: 200px;
    padding: 5px;
    width: 125px;
  }
  .color-swatch ul {
    margin: 5px 5px 5px 15px;
  }
  .color-swatch ul li {
    background: none;
    list-style: disc;
    padding-left: 0;
  }
  </style>
</asp:Content>
<asp:Content runat="server" ContentPlaceHolderID="MainContent">
<div id="styleGuide">
  <div id="content" class="container_12">
    <div id="pageHeader">
      <h1>Style Guide</h1>
      <span id="ctl00_MainContent_ctl00_BreadcrumbPath" class="breadcrumb">
      <span>
      <a class="root" href="/Default.aspx" title="Technology Toolbox home page">
      Home</a></span> <span class="separator">&gt; </span>
      <span class="current">Style Guide</span> </span></div>
    <div id="contentMain" class="grid_7">
      <h2>Heading 2</h2>
      <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus 
      eleifend purus at est hendrerit consectetur pretium turpis tempus. Maecenas 
      pulvinar mattis fringilla. Nam ornare sollicitudin dui sit amet pulvinar. 
      Proin nunc lacus, aliquet in sagittis rhoncus, suscipit sit amet leo.</p>
      <p>Ut euismod lorem non nibh varius in lobortis enim aliquam. Aliquam 
      sed semper lacus. Aliquam sed tellus nec nibh dignissim consectetur 
      sit amet ac dolor. Donec auctor hendrerit vehicula. Maecenas a sollicitudin 
      lorem. Mauris id nibh ut urna cursus auctor. Vestibulum scelerisque, 
      nisi sit amet vulputate venenatis, ipsum leo vulputate dolor, sit amet 
      mattis ligula urna at leo.</p>
      <h3>Heading 3</h3>
      <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce nec 
      diam nec sapien luctus iaculis. Nam neque. Mauris sit amet quam. Nunc 
      fermentum, erat in pellentesque porttitor,
      <span class="highlightRed">magna mauris consectetur</span> nibh, et 
      vestibulum augue urna in felis. Integer et sapien. Mauris neque nibh, 
      rutrum sed, pharetra ut, suscipit in, lacus. Nullam mauris. Praesent 
      fringilla sollicitudin nunc. Nullam ipsum neque, malesuada vitae, blandit 
      sed, tincidunt sed, mauris. Nullam blandit.
      <span class="highlightYellow">Suspendisse sed turpis</span> sit amet 
      diam tincidunt imperdiet.</p>
      <blockquote class="directQuote errorMessage">
        [Direct quote, error message] Pellentesque habitant morbi tristique 
        senectus et netus et malesuada fames ac turpis egestas.</blockquote>
      <h4>Heading 4</h4>
      <p>Fusce blandit interdum pretium. Lorem ipsum dolor sit amet, consectetur 
      adipiscing elit. Phasellus ut augue eget arcu blandit vestibulum. Maecenas 
      a nisi libero, et elementum purus. </p>
      <h5>Heading 5</h5>
      <p>Nullam consequat urna mattis eros sodales aliquet. Pellentesque habitant 
      morbi tristique senectus et netus et malesuada fames ac turpis egestas. 
      Integer eget purus velit, nec pellentesque mauris. Suspendisse et leo 
      id enim scelerisque hendrerit.</p>
      <h6>Heading 6</h6>
      <p>Class aptent taciti sociosqu ad litora torquent per conubia nostra, 
      per inceptos himenaeos. Donec viverra nunc eu ligula bibendum mattis. 
      Pellentesque a tellus ac nunc vulputate commodo a consectetur nibh. 
      Proin sit amet velit diam, in vulputate augue.</p>
      <p>Unordered list:</p>
      <ul>
        <li>Fusce viverra velit eu pede. </li>
        <li>Integer ornare nisl ac lorem. </li>
        <li>Sed varius pretium mauris. </li>
        <li>Aenean id metus vitae enim consequat lobortis.
        <ul>
          <li>Quisque sit amet nisi sit amet ipsum scelerisque pharetra.</li>
          <li>Praesent interdum arcu eget eros condimentum eu sagittis 
          odio blandit.</li>
        </ul>
        </li>
        <li>Curabitur scelerisque augue quis quam. </li>
        <li>Donec elementum est in quam auctor viverra. </li>
      </ul>
      <p>Ordered list:</p>
      <ol>
        <li>Etiam sodales eros vel felis lacinia auctor.</li>
        <li>In consequat est eget tortor dignissim non pellentesque augue 
        faucibus.</li>
        <li>Proin non diam nisl, eu cursus leo.</li>
        <li>Praesent nec sem ipsum, in euismod odio.<ol>
          <li>Cras nec arcu eros, vitae blandit purus.</li>
          <li>Donec viverra tortor ut nisi viverra placerat eu vel nulla.
          <ol>
            <li>Quisque sit amet nisi sit amet ipsum scelerisque pharetra.</li>
            <li>Praesent interdum arcu eget eros condimentum eu sagittis 
            odio blandit.</li>
          </ol>
          </li>
          <li>Quisque a tellus lectus, non vulputate ipsum.</li>
          <li>Mauris aliquet leo ac mauris convallis eget sagittis mauris 
          bibendum.</li>
        </ol>
        </li>
      </ol>
      <p>Ordered list within an unordered list:</p>
      <ul>
        <li>Etiam sodales eros vel felis lacinia auctor.</li>
        <li>In consequat est eget tortor dignissim non pellentesque augue 
        faucibus.</li>
        <li>Proin non diam nisl, eu cursus leo.</li>
        <li>Praesent nec sem ipsum, in euismod odio.<ol>
          <li>Cras nec arcu eros, vitae blandit purus.</li>
          <li>Donec viverra tortor ut nisi viverra placerat eu vel nulla.</li>
          <li>Quisque a tellus lectus, non vulputate ipsum.</li>
          <li>Mauris aliquet leo ac mauris convallis eget sagittis mauris 
          bibendum.</li>
        </ol>
        </li>
      </ul>
      <p>Image:</p>
      <div class="image">
        <img alt="" height="99" src="http://blogs.msdn.com/photos/jjameson/images/8563260/500x99.aspx" title="" width="500" />
        <div class="caption">
          Figure 1: Scheduled tasks for backing up databases</div>
        <div class="imageLink">
          <a href="http://blogs.msdn.com/photos/jjameson/images/8563260/original.aspx" target="_blank">
          See full-sized image.</a> </div>
      </div>
      <p>Table:</p>
      <table cellspacing="0" class="accent1" width="100%">
        <caption>MOSS 2007 Feature Definitions</caption>
        <thead>
          <tr>
            <th>Feature Id</th>
            <th>Display Name</th>
            <th>Scope</th>
            <th>Solution</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>99ee0928-7342-4739-865d-35b61ea4eaf0</td>
            <td>BDCAdminUILinks*</td>
            <td>Farm</td>
            <td>Microsoft.Office.Excel.Server</td>
          </tr>
          <tr>
            <td>cdfa39c6-6413-4508-bccf-bf30368472b3</td>
            <td>DataConnectionLibraryStapling**</td>
            <td>Farm</td>
            <td>Microsoft.Office.Excel.Server</td>
          </tr>
          <tr>
            <td>97a2485f-ef4b-401f-9167-fa4fe177c6f6</td>
            <td>BaseSiteStapling</td>
            <td>Farm</td>
            <td>&nbsp;</td>
          </tr>
        </tbody>
      </table>
      <div class="footnotes">
        <div class="footnote">
          * - Some footnote</div>
        <div class="footnote">
          ** - Another footnote</div>
      </div>
      <p>Reference:</p>
      <div class="reference">
        <cite>Napier, Bryan (2007). SPLimitedWebPartManager Memory Leak? 
        .. of ones and zeros.. 2007-06-05.</cite>
        <div class="referenceLink">
          <a href="http://blog.ofonesandzeros.com/2007/06/05/splimitedwebpartmanager-memory-leak/">
          http://blog.ofonesandzeros.com/2007/06/05/splimitedwebpartmanager-memory-leak/</a></div>
      </div>
      <p>Note:</p>
      <blockquote class="note">
        <div class="noteTitle">
          <strong>Note</strong></div>
        <div>
          [Note text]</div>
      </blockquote>
      <p>Important note:</p>
      <blockquote class="note important">
        <div class="noteTitle">
          <strong>Important</strong></div>
        <div>
          [Note text]</div>
      </blockquote>
      <p>Warning:</p>
      <blockquote class="note warning">
        <div class="noteTitle">
          <strong>Warning</strong></div>
        <div>
          [Warning text]</div>
      </blockquote>
      <p>Update:</p>
      <blockquote class="note update">
        <div class="noteTitle">
          <strong>Update (2010-05-21)</strong></div>
        <div>
          Something added or changed...</div>
      </blockquote>
      <p>Definition list:</p>
      <dl>
        <dt>Major Version</dt>
        <dd>Manually incremented for major releases, such as adding many 
        new features to the solution. </dd>
        <dt>Minor Version</dt>
        <dd>Manually incremented for minor releases, such as introducing 
        small changes to existing features. </dd>
        <dt>Build Number </dt>
        <dd>Typically incremented automatically as part of every build performed 
        on the Build Server. This allows each build to be tracked and tested.
        </dd>
        <dt>Revision </dt>
        <dd>Incremented for QFEs (a.k.a. "hotfixes" or patches) to builds 
        released into the Production environment (PROD). This is set to 
        zero for the initial release of any major/minor version of the solution.</dd>
      </dl>
      <p>Conversation:</p>
      <ol class="conversation">
        <li><cite>Dr. Jekyll</cite>
        <blockquote class="directQuote">
          <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. 
          Nullam volutpat massa a sapien pharetra a dignissim metus rhoncus.</p>
          <p>Morbi quis ipsum nec nunc mollis accumsan sit amet eu eros. 
          Mauris porttitor posuere nunc sed tincidunt. Nulla facilisi. 
          Duis tortor lacus, auctor et tempor et, porta at tortor. Mauris 
          semper orci et nisl suscipit sit amet lacinia enim facilisis. 
          Ut orci lorem, gravida vel dignissim ac, tincidunt sed enim. 
          In non est at tortor faucibus ullamcorper ut et enim.</p>
        </blockquote>
        </li>
        <li><cite>Mr. Hyde</cite>
        <blockquote class="directQuote">
          <p>Ut vestibulum dictum nisl vel tempus. In sit amet sapien 
          lectus, luctus varius dolor. Praesent eu nunc est. Maecenas 
          pellentesque vulputate fermentum. Morbi venenatis quam enim, 
          nec porta magna. Mauris rhoncus porttitor sapien, a gravida 
          turpis porttitor at. Aliquam imperdiet eros eget erat hendrerit 
          sollicitudin consectetur magna ornare.</p>
        </blockquote>
        </li>
        <li><cite>Dr. Jekyll</cite>
        <blockquote class="directQuote">
          <p>Nulla nec tellus dui. Aliquam ut augue mi, eget convallis 
          augue.</p>
        </blockquote>
        </li>
      </ol>
      <p>Console block (simple - input only):</p>
      <div class="consoleBlock">
        <kbd>stsadm -o addsolution -filename Fabrikam.Project1.PublishingLayouts.wsp</kbd><br/>
        <kbd>stsadm -o deploysolution -name Fabrikam.Project1.PublishingLayouts -url
          <a href="http://foobar/">http://foobar/</a> -local</kbd>
      </div>
      <p>Console block (complex - keyboard input and sample output):</p>
      <div class="consoleBlock">
        <p>C:\NotBackedUp\Fabrikam\Project1\Main\PublicationLibrary\DeploymentFiles\Scripts&gt;<kbd>"Activate 
        Feature.cmd"</kbd></p>
        <p><samp>Activating Fabrikam.Project1.PublicationLibrary on url 
        - http://project1-local</samp></p>
        <p><samp>Dependency feature 'Fabrikam.Project1.PublicationContentTypes' 
        (id: 9f5c14f1-cf58-47c7-bbba-da9a8637deab) is not properly scoped 
        for feature 'Fabrikam.Project1.PublicationLibrary' (id: 49b204d0-7e35-4460-a691-a7d481c463b4). 
        Its scope 'Site' must be equal to or higher than 'WebApplication'.</samp></p>
      </div>
      <p>WinDbg session:</p>
      <div class="consoleBlock">
        <pre>0:023&gt; <kbd>ld mscorwks</kbd>
<samp>Symbols loaded for mscorwks</samp>
0:023&gt; <kbd>.cordll -lp C:\Windows\Microsoft.NET\Framework\v2.0.50727</kbd>
<samp>CLR DLL status: No load attempts</samp>
0:023&gt; <kbd>!threads</kbd>
<samp>Index TID   TEB    StackBase   StackLimit   DeAlloc   StackSize   ThreadProc 
0 00000dec 0x7ffdf000 0x00110000 0x00105000 0x000d0000 0x0000b000 0x0 
1 0000156c 0x7ffde000 0x01550000 0x0154e000 0x01510000 0x00002000 0x0 
2 0000144c 0x7ffdd000 0x00df0000 0x00dee000 0x00db0000 0x00002000 0x0
...</samp></pre>
      </div>
      <p>Log excerpt:</p>
      <div class="logExcerpt">
        <pre><samp>04/08/2008 ... Applying template &quot;TfsLite.stp&quot; to web at URL &quot;http://wss-dev/Test&quot;.   
04/08/2008 ... Failed to get the site template for language 1033, search key 'TfsLite.stp'. This warning is expected when provisioning from a custom web template.
04/08/2008 ... Marking web-scoped features active from manifest at URL &quot;http://wss-dev/Test&quot;
04/08/2008 ... Failed to mark site-scoped features active in site 'http://wss-dev/Test'.
04/08/2008 ... Failed to apply template &quot;TfsLite.stp&quot; to web at URL &quot;http://wss-dev/Test&quot;.
04/08/2008 ... Failed to apply template &quot;TfsLite.stp&quot; to web at URL &quot;http://wss-dev/Test&quot;, error The template you have chosen is invalid or cannot be found. 0x81071e44
04/08/2008 ... The template you have chosen is invalid or cannot be found.</samp></pre>
      </div>
      <p>Code block:</p>
      <div class="codeBlock">
        <pre><code><span style="color: rgb(0, 0, 255);">static</span> <span style="color: rgb(0, 0, 255);">void</span> Main(<span style="color: rgb(0, 0, 255);">string</span>[] args)
{
    <span style="color: rgb(43, 145, 175);">Uri</span> siteUrl = <span style="color: rgb(0, 0, 255);">new</span> <span style="color: rgb(43, 145, 175);">Uri</span>(<span style="color: rgb(163, 21, 21);">&quot;http://foobar/sites/Migration&quot;</span>);

    SPWebApplication application = SPWebApplication.Lookup(siteUrl);

    <span style="color: rgb(0, 0, 255);">foreach</span> (SPPolicyRole policyRole <span style="color: rgb(0, 0, 255);">in</span> application.PolicyRoles)
    {
        <span style="color: rgb(43, 145, 175);">Console</span>.WriteLine(policyRole.Name);
    }
}</code></pre>
      </div>
      <p>PowerShell script:</p>
      <div class="codeBlock">
        <pre><code><span class="comment"># Adds a &quot;Wiki&quot; library to a SharePoint site</span>

<span class="keyword">function</span> <span class="commandArgument">AddWikiLibrary</span>(
    <span class="userType">[Microsoft.SharePoint.SPweb]</span> <span class="variable">$web</span><span class="operator">,</span>
    <span class="variable">$libraryName</span><span class="operator">,</span>
    <span class="variable">$libraryDescription</span>)
{
    <span class="command">Write-Debug</span> <span class="string">&quot;Adding wiki library ($libraryName) to site ($($web.Url))...&quot;</span>
                    
    <span class="variable">$listId</span> <span class="operator">=</span> <span class="variable">$web</span><span class="operator">.</span><span class="member">Lists</span><span class="operator">.</span><span class="member">Add</span>(
        <span class="variable">$name</span><span class="operator">,</span>
        <span class="variable">$description</span><span class="operator">,</span>
        <span class="userType">[Microsoft.SharePoint.SPListTemplateType]</span><span class="operator">::</span><span class="member">WebPageLibrary</span>)
    
    <span class="variable">$wikiLibrary</span> <span class="operator">=</span> <span class="variable">$web</span><span class="operator">.</span><span class="member">Lists</span><span class="operator">[</span><span class="variable">$listId</span><span class="operator">]</span>
    
    <span class="variable">$null</span> <span class="operator">=</span> <span class="userType">[Microsoft.SharePoint.Utilities.SPUtility]</span><span class="operator">::</span><span class="member">AddDefaultWikiContent</span>(
        <span class="variable">$wikiLibrary</span>)
}

<span class="variable">$name</span> <span class="operator">=</span> <span class="string">&quot;Team Wiki&quot;</span>

<span class="variable">$description</span> <span class="operator">=</span> <span class="string">&quot;Share knowledge for a Team Project by adding or editing&quot;</span> `
    <span class="operator">+</span> <span class="string">&quot; content in this wiki&quot;</span>

<span class="variable">$sitesToUpgrade</span> <span class="operator">=</span>
   @(
  <span class="string">&quot;http://cyclops/sites/AdventureWorks&quot;</span><span class="operator">,</span>
  <span class="string">&quot;http://cyclops/sites/Demo&quot;</span><span class="operator">,</span>
  <span class="string">&quot;http://cyclops/sites/Toolbox&quot;</span>
   )
   
<span class="variable">$sitesToUpgrade</span> <span class="operator">|</span>
    <span class="command">ForEach-Object</span> {
        <span class="variable">$DebugPreference</span> <span class="operator">=</span> <span class="string">&quot;SilentlyContinue&quot;</span>
        <span class="variable">$web</span> <span class="operator">=</span> <span class="command">Get-SPWeb</span> <span class="variable">$_</span>

        <span class="variable">$DebugPreference</span> <span class="operator">=</span> <span class="string">&quot;Continue&quot;</span>
        <span class="command">AddWikiLibrary</span> <span class="variable">$web</span> <span class="variable">$name</span> <span class="variable">$description</span>
        <span class="variable">$web</span><span class="operator">.</span><span class="member">Dispose</span>()
    }</code></pre>
      </div>
      <h2>Colors</h2>
      <p>The following swatches show all of the colors used throughout the 
      site (with the exception of color syntax highlighting in code samples). 
      However they do not necessarily list all instances where the colors 
      are used.</p>
      <div class="color-swatch" style="background-color: #4f4f4f; color: #fff">
        <strong>#4f4f4f</strong><ul>
          <li>body text</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #959595; color: #000">
        <strong>#959595</strong><ul>
          <li>disabled button</li>
          <li>fieldset border</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #eee; color: #000">
        <strong>#eee</strong><ul>
          <li>disabled button background</li>
          <li>fieldset background</li>
          <li>code and console blocks background</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #77c856; color: #fff">
        <strong>#77c856</strong><ul>
          <li>h1 background</li>
          <li>accent borders</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #3c78c3; color: #fff">
        <strong>#3c78c3</strong><ul>
          <li>hyperlinks</li>
          <li>horizontal rules</li>
          <li>comment borders</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #1e4173; color: #fff">
        <strong>#1e4173</strong><ul>
          <li>buttons</li>
          <li>h2, h3, h4</li>
          <li>dl dt</li>
          <li>table cell borders</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #c8d7eb; color: #000">
        <strong>#c8d7eb</strong><ul>
          <li>h2 border, h3 border</li>
          <li>comments by blog author</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #bd1c1c; color: #fff">
        <strong>#bd1c1c</strong><ul>
          <li>error messages</li>
          <li>warnings</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #fbeded; color: #000">
        <strong>#fbeded</strong><ul>
          <li>warning background</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #ff9999; color: #000">
        <strong>#ff9999</strong><ul>
          <li>highlight red</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #ffff99; color: #000">
        <strong>#ffff99</strong><ul>
          <li>highlight yellow</li>
        </ul>
      </div>
      <div class="color-swatch" style="background-color: #ffffcc; color: #000">
        <strong>#ffffcc</strong><ul>
          <li>required field background</li>
        </ul>
      </div>
    </div>
    <div id="contentSub" class="grid_5">
      <img alt="Gears icon" src="Images/icon-gears-347x346.jpg" /> </div>
  </div>
</div>
</asp:Content>
```

> **Note**
>
> The `color-swatch` CSS rules at the top of Style-Guide.aspx are not kept in the "main" cascading style sheet since these rules are applicable only to the Style Guide itself. Also note that the default master page (specified in the Web.config file) defines the `<link>` element that references the CSS file.

Figure 1 shows the corresponding page rendered in the browser. [Note: This is  not intended to be an eye chart. Normally, I would trim images like this before  including them in a post, but in this case I wanted to be sure you could see the  page in its entirety (by clicking the **See full-sized image** link  below.)]

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Style-Guide.png"
alt="Style Guide for TechnologyToolbox.com"
height="600"
width="110"
title="Figure 1: Style Guide for TechnologyToolbox.com" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Style-Guide.png)

### Color palette

Another benefit of a Style Guide is the ability to quickly find a color from  the "approved" color palette for the website.

I used to insert a "glossary" in the CSS (using comments) of the various hex  color codes specified in CSS rules so I could identify different colors later on  when adding new CSS rules. However, adequately describing the various colors gets  to be somewhat challenging and nebulous (e.g. "Securitas dark gray" vs. "Securitas  medium gray").

Now I simply create a bunch of "color swatches" in the Style Guide that show  the various colors and provide descriptions for where the colors are used, as shown  below.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Color-Palette.png"
alt="Technology Toolbox color palette"
height="600"
width="518"
title="Figure 2: Technology Toolbox color palette" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Color-Palette.png)

### Sample style guide for SharePoint 2010

Here is the HTML for a sample Style Guide that I developed for a SharePoint 2010  project earlier this year. I found this to be especially helpful in customizing  the appearance of the out-of-the-box styles users can specify in the SharePoint  Rich Text Editor (e.g. "Colored Heading 1") as well as the multitude of different  table formatting options (e.g. different odd/even rows).

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>Sample Style Guide for SharePoint 2010</title>
</head>
<body>
    <h1>
        Heading 1</h1>
    <h2>
        Heading 2</h2>
    <p class="ms-rteElement-P ms-rteStyle-Tagline">
        Tag Line - Nunc ornare rutrum odio, ac fringilla urna euismod sed</p>
    <p class="ms-rteElement-P ms-rteStyle-Byline">
        By Line - Aliquam Ullamcorper</p>
    <p class="ms-rteElement-P">
        Lorem ipsum dolor sit amet, <span>consectetur adipiscing elit</span>. Nulla bibendum
        justo in mauris convallis ac gravida tortor laoreet. <a href="http://blogs.msdn.com/jjameson">
            Vestibulum sapien velit</a>, ornare ac hendrerit vitae, condimentum a erat.
        Duis feugiat, est non vulputate consectetur, lacus felis placerat nunc, et dignissim
        urna neque a lectus. Fusce sit amet erat lectus, sed semper enim. Nulla tempor sem
        vel arcu luctus id luctus orci ultrices. Nulla consequat lectus eu urna iaculis
        a pretium leo adipiscing. Suspendisse tristique volutpat dolor ut adipiscing. Duis
        sem dui, pretium in auctor ut, venenatis vitae est. <span class="ms-rteStyle-Highlight">
            Highlight</span></p>
    <p class="ms-rteElement-P ms-rteStyle-Comment">
        <span>Comment - Morbi ornare mollis lacus, sed tempor lorem pellentesque ut. Vestibulum
            adipiscing lacus non nunc vestibulum viverra. Nulla facilisi. Donec turpis metus,
            pretium et volutpat et, lobortis eget risus</span></p>
    <p class="ms-rteElement-P ms-rteStyle-References">
        <span>References - Etiam dignissim luctus luctus. Nullam at mi adipiscing erat euismod
            facilisis.</span></p>
    <h3>
        Heading 3</h3>
    <h4>
        Heading 4</h4>
    <p>
        Bulleted List:</p>
    <ul>
        <li>Item 1
            <ul>
                <li>Item 1a</li>
                <li>Item 1b</li>
            </ul>
        </li>
        <li>Item 2
            <ul>
                <li>Item 2a</li>
                <li>Item 2b</li>
            </ul>
        </li>
        <li>Item 3</li>
    </ul>
    <p>
        Ordered List:</p>
    <ol>
        <li>Item 1
            <ol>
                <li>Item 1a</li>
                <li>Item 1b</li>
            </ol>
        </li>
        <li>Item 2<ol>
            <li>Item 2a</li>
            <li>Item 2b</li>
        </ol>
        </li>
    </ol>
    <ol>
        <li>Item 3</li>
    </ol>
    <h1 class="ms-rteElement-H1B">
        Colored Heading 1</h1>
    <div>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla bibendum justo in
        mauris convallis ac gravida tortor laoreet. Vestibulum sapien velit, ornare ac hendrerit
        vitae, condimentum a erat. Duis feugiat, est non vulputate consectetur, lacus felis
        placerat nunc, et dignissim urna neque a lectus. Fusce sit amet erat lectus, sed
        semper enim. Nulla tempor sem vel arcu luctus id luctus orci ultrices. Nulla consequat
        lectus eu urna iaculis a pretium leo adipiscing. Suspendisse tristique volutpat
        dolor ut adipiscing. Duis sem dui, pretium in auctor ut, venenatis vitae est.</div>
    <h2 class="ms-rteElement-H2B">
        Colored Heading 2</h2>
    <div>
        Curabitur augue justo, blandit at dictum et, varius nec metus. Aenean eget eros
        a risus pellentesque tincidunt vitae a arcu. Quisque erat nulla, luctus a semper
        in, sodales vel arcu.</div>
    <div>
        <img alt="" src="/SiteCollectionImages/PR.gif" style="margin: 5px" /><br />
        <br />
        <span class="ms-rteStyle-Caption">Caption - </span><span class="ms-rteStyle-Caption">
            Nam lacus leo, luctus et vulputate id, laoreet nec leo</span></div>
    <h3 class="ms-rteElement-H3B">
        Colored Heading 3</h3>
    <p class="ms-rteElement-P">
        Etiam vestibulum luctus mauris eget vestibulum. Etiam laoreet consectetur quam,
        at lobortis nibh tristique ac. Donec et quam nec mi mattis tempus interdum in sem.
        Pellentesque sagittis, ipsum vel tincidunt sollicitudin, lectus lacus volutpat augue,
        eget malesuada nibh leo quis libero. Nulla lobortis aliquet commodo. Morbi ac egestas
        dui. Proin at mattis diam. Fusce lacus nisl, fermentum at ultrices non, volutpat
        vehicula mi. Nulla ac urna vel nunc gravida ultricies sed ut mi. Maecenas ipsum
        risus, vehicula sed varius ut, accumsan non quam. Quisque venenatis purus nec tellus
        elementum in aliquam erat hendrerit. Cras lacus orci, varius fringilla hendrerit
        non, dictum eget quam. Phasellus ullamcorper mauris id metus commodo ut tempus nisl
        congue.</p>
    <div class="ms-rteElement-Callout1">
        Callout 1 - Nunc tristique aliquam nunc, et mollis dui mollis sit amet. Proin sed
        arcu a ipsum consequat volutpat. Curabitur laoreet nibh at metus posuere sagittis.
    </div>
    <p class="ms-rteElement-P">
        Sed eget lectus a mauris sodales rhoncus auctor et orci. Ut ac dui ligula, sed convallis
        nulla. Maecenas sodales laoreet sapien. Donec rutrum, nulla non tempor eleifend,
        nulla risus pretium mauris, in porta quam leo vel massa. Pellentesque eu dui non
        risus tempus pellentesque nec at risus. Donec luctus porttitor eros eu posuere.
        Praesent imperdiet placerat tellus ut fringilla. Sed feugiat nulla vehicula eros
        pulvinar sed hendrerit est vestibulum. Nam dignissim vehicula massa a ornare. Integer
        quis mi a nibh rutrum tincidunt et ac nulla. Aenean aliquet elementum dolor, et
        luctus nibh semper et. Suspendisse et leo velit. Sed sollicitudin vulputate arcu,
        non lobortis eros condimentum at. Vivamus varius congue luctus.</p>
    <p class="ms-rteElement-P">
        Aenean tincidunt accumsan mauris, non convallis est ornare in. Vestibulum et venenatis
        orci. Aenean ut nulla odio, et porta leo. Fusce accumsan ante sed nibh tempor ultrices.
        Suspendisse eget orci dui, ac posuere tortor. Aliquam augue erat, adipiscing eget
        venenatis at, euismod id nisl. Proin commodo, quam vel aliquet tempus, dolor arcu
        placerat arcu, a consequat urna metus vel libero. Morbi fermentum libero quis nibh
        rhoncus pharetra. Pellentesque accumsan libero eget velit imperdiet tristique. Vestibulum
        eu tortor faucibus mi mollis aliquet a quis nunc. Pellentesque dictum mauris non
        mi porta aliquet. Aenean vestibulum nisi magna. Donec suscipit sapien in leo hendrerit
        elementum. Integer eget urna nec enim elementum ultricies. Nulla sit amet turpis
        sed lacus tristique porta sagittis eu purus. Duis tincidunt, nibh et blandit ultrices,
        nisl sapien posuere metus, non fermentum arcu nisl sed mauris.</p>
    <div class="ms-rteElement-Callout2">
        Callout 2 - Vestibulum tristique bibendum odio, ut mattis eros ultrices vel. In
        pretium vestibulum enim porttitor pulvinar. Duis tellus lacus, aliquet eget tincidunt
        ac, rutrum eget odio.
    </div>
    <p class="ms-rteElement-P">
        Nam elementum, arcu sed convallis consectetur, diam est consectetur nulla, at condimentum
        turpis eros eu lacus. Integer sollicitudin, tellus in sollicitudin ullamcorper,
        erat libero commodo libero, nec gravida lacus ante vel diam. Nunc dolor nunc, condimentum
        ultrices lobortis id, gravida eu augue. Nam sed arcu in risus dapibus pretium non
        sed lacus. Proin eu imperdiet sem. In elit arcu, volutpat et auctor quis, porttitor
        eu elit. In in massa nec quam posuere gravida in eu ipsum. Aliquam sapien ipsum,
        porttitor in dapibus mattis, ullamcorper et magna. Pellentesque egestas placerat
        lacus, et pulvinar metus luctus sit amet. Integer blandit, nisl eget egestas ullamcorper,
        ante dolor feugiat risus, non fringilla diam leo et orci. Phasellus eleifend, quam
        sed vestibulum feugiat, odio massa interdum est, in ullamcorper turpis quam vel
        lorem. Suspendisse faucibus tellus vitae quam dignissim scelerisque. Suspendisse
        enim nisl, imperdiet a condimentum in, ultrices ac urna. Duis faucibus arcu vel
        nisl lacinia non ullamcorper lectus fermentum. Donec non enim in nulla iaculis ornare
        cursus ut lacus.</p>
    <p class="ms-rteElement-P">
        Cras ultricies viverra magna in viverra. Cras risus nisi, tempus nec luctus vel,
        blandit eget urna. Nam a cursus felis. In ultricies purus quis lorem hendrerit interdum
        non facilisis tellus. Donec consequat eros at urna iaculis venenatis. Duis vehicula
        rutrum diam, ut fringilla sem vestibulum eget. Nam eu risus nulla. Donec vel est
        vestibulum turpis vestibulum euismod quis laoreet metus. Donec at velit nunc, quis
        mattis augue. Cras dictum, urna non laoreet pharetra, massa dolor fermentum metus,
        a tempus velit velit nec massa.</p>
    <div class="ms-rteElement-Callout3">
        Callout 3 -&#160;&#160;Donec consectetur sollicitudin diam a dapibus. Nulla facilisi.
        Etiam suscipit turpis mi. Morbi eu ipsum quam, eget elementum elit. Integer nisl
        sapien, malesuada ac vulputate et, accumsan vitae elit. Etiam nec odio nisi.
    </div>
    <p class="ms-rteElement-P">
        In hac habitasse platea dictumst. Vestibulum ante ipsum primis in faucibus orci
        luctus et ultrices posuere cubilia Curae; Donec eget nunc id sem condimentum luctus.
        Integer iaculis iaculis sem sed condimentum. Aliquam erat volutpat. Nunc vulputate
        purus vitae ipsum vestibulum non suscipit magna varius. Donec libero urna, accumsan
        nec mattis viverra, viverra at erat. Donec egestas congue turpis et consequat. Nam
        varius interdum nisi, ac fermentum sem aliquam non. In ut nisl vitae urna euismod
        pellentesque et sit amet sem. Sed sed felis eget ante imperdiet sodales ut eget
        magna. Aliquam erat volutpat. Sed eget nibh magna. Phasellus blandit malesuada malesuada.
        Nulla facilisi. Duis interdum orci et urna vulputate hendrerit. Fusce nulla enim,
        tincidunt sit amet imperdiet tristique, tempor sed ligula.</p>
    <div class="ms-rteElement-Callout4">
        Callout 4 - Curabitur scelerisque turpis sit amet lacus dignissim lacinia. Sed nibh
        sem, iaculis id auctor non, posuere sed sapien. Vivamus nunc erat, eleifend in suscipit
        eget, tempor sed turpis.</div>
    <p class="ms-rteElement-P">
        Aenean tincidunt accumsan mauris, non convallis est ornare in. Vestibulum et venenatis
        orci. Aenean ut nulla odio, et porta leo. Fusce accumsan ante sed nibh tempor ultrices.</p>
    <h4 class="ms-rteElement-H4B">
        Colored Heading 4</h4>
    <h2>
        Table Format Default-A</h2>
    <table class="ms-rteTable-default" style="width: 100%">
        <tr class="ms-rteTableEvenRow-default">
            <td class="ms-rteTableEvenCol-default">
                &#160;
            </td>
            <td class="ms-rteTableOddCol-default">
                Jan
            </td>
            <td class="ms-rteTableEvenCol-default">
                Feb
            </td>
            <td class="ms-rteTableOddCol-default">
                Mar
            </td>
            <td class="ms-rteTableEvenCol-default">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-default">
            <td class="ms-rteTableEvenCol-default">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-default">
                7
            </td>
            <td class="ms-rteTableOddCol-default">
                5
            </td>
            <td class="ms-rteTableEvenCol-default">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-default">
            <td class="ms-rteTableEvenCol-default">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-default">
                4
            </td>
            <td class="ms-rteTableOddCol-default">
                7
            </td>
            <td class="ms-rteTableEvenCol-default">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-default">
            <td class="ms-rteTableEvenCol-default">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-default">
                7
            </td>
            <td class="ms-rteTableOddCol-default">
                9
            </td>
            <td class="ms-rteTableEvenCol-default">
                24
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-default">
            <td class="ms-rteTableEvenCol-default">
                &#160;Total
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;21
            </td>
            <td class="ms-rteTableEvenCol-default">
                18
            </td>
            <td class="ms-rteTableOddCol-default">
                21
            </td>
            <td class="ms-rteTableEvenCol-default">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format Default-B</h2>
    <table class="ms-rteTable-default" style="width: 100%">
        <tr class="ms-rteTableHeaderRow-default">
            <td class="ms-rteTableHeaderEvenCol-default">
                &#160;
            </td>
            <td class="ms-rteTableHeaderOddCol-default">
                Jan
            </td>
            <td class="ms-rteTableHeaderEvenCol-default">
                Feb
            </td>
            <td class="ms-rteTableHeaderOddCol-default">
                Mar
            </td>
            <td class="ms-rteTableHeaderEvenCol-default">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-default">
            <td class="ms-rteTableEvenCol-default">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-default">
                7
            </td>
            <td class="ms-rteTableOddCol-default">
                5
            </td>
            <td class="ms-rteTableEvenCol-default">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-default">
            <td class="ms-rteTableEvenCol-default">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-default">
                4
            </td>
            <td class="ms-rteTableOddCol-default">
                7
            </td>
            <td class="ms-rteTableEvenCol-default">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-default">
            <td class="ms-rteTableEvenCol-default">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-default">
                7
            </td>
            <td class="ms-rteTableOddCol-default">
                9
            </td>
            <td class="ms-rteTableEvenCol-default">
                24
            </td>
        </tr>
        <tr class="ms-rteTableFooterRow-default">
            <td class="ms-rteTableFooterEvenCol-default">
                &#160;Total
            </td>
            <td class="ms-rteTableFooterOddCol-default">
                &#160;21
            </td>
            <td class="ms-rteTableFooterEvenCol-default">
                18
            </td>
            <td class="ms-rteTableFooterOddCol-default">
                21
            </td>
            <td class="ms-rteTableFooterEvenCol-default">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format Default-C</h2>
    <table class="ms-rteTable-default" style="width: 100%">
        <tr class="ms-rteTableHeaderRow-default">
            <td class="ms-rteTableHeaderFirstCol-default">
                &#160;
            </td>
            <td class="ms-rteTableHeaderOddCol-default">
                Jan
            </td>
            <td class="ms-rteTableHeaderEvenCol-default">
                Feb
            </td>
            <td class="ms-rteTableHeaderOddCol-default">
                Mar
            </td>
            <td class="ms-rteTableHeaderLastCol-default">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-default">
            <td class="ms-rteTableFirstCol-default">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-default">
                7
            </td>
            <td class="ms-rteTableOddCol-default">
                5
            </td>
            <td class="ms-rteTableLastCol-default">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-default">
            <td class="ms-rteTableFirstCol-default">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-default">
                4
            </td>
            <td class="ms-rteTableOddCol-default">
                7
            </td>
            <td class="ms-rteTableLastCol-default">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-default">
            <td class="ms-rteTableFirstCol-default">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-default">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-default">
                7
            </td>
            <td class="ms-rteTableOddCol-default">
                9
            </td>
            <td class="ms-rteTableLastCol-default">
                24
            </td>
        </tr>
        <tr class="ms-rteTableFooterRow-default">
            <td class="ms-rteTableFooterFirstCol-default">
                &#160;Total
            </td>
            <td class="ms-rteTableFooterOddCol-default">
                &#160;21
            </td>
            <td class="ms-rteTableFooterEvenCol-default">
                18
            </td>
            <td class="ms-rteTableFooterOddCol-default">
                21
            </td>
            <td class="ms-rteTableFooterLastCol-default">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format 1-A</h2>
    <table class="ms-rteTable-0" style="width: 100%">
        <tr class="ms-rteTableEvenRow-0">
            <td class="ms-rteTableEvenCol-0">
                &#160;
            </td>
            <td class="ms-rteTableOddCol-0">
                Jan
            </td>
            <td class="ms-rteTableEvenCol-0">
                Feb
            </td>
            <td class="ms-rteTableOddCol-0">
                Mar
            </td>
            <td class="ms-rteTableEvenCol-0">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-0">
            <td class="ms-rteTableEvenCol-0">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-0">
                7
            </td>
            <td class="ms-rteTableOddCol-0">
                5
            </td>
            <td class="ms-rteTableEvenCol-0">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-0">
            <td class="ms-rteTableEvenCol-0">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-0">
                4
            </td>
            <td class="ms-rteTableOddCol-0">
                7
            </td>
            <td class="ms-rteTableEvenCol-0">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-0">
            <td class="ms-rteTableEvenCol-0">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-0">
                7
            </td>
            <td class="ms-rteTableOddCol-0">
                9
            </td>
            <td class="ms-rteTableEvenCol-0">
                24
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-0">
            <td class="ms-rteTableEvenCol-0">
                &#160;Total
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;21
            </td>
            <td class="ms-rteTableEvenCol-0">
                18
            </td>
            <td class="ms-rteTableOddCol-0">
                21
            </td>
            <td class="ms-rteTableEvenCol-0">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format 1-B</h2>
    <table class="ms-rteTable-0" style="width: 100%">
        <tr class="ms-rteTableHeaderRow-0">
            <td class="ms-rteTableHeaderEvenCol-0">
                &#160;
            </td>
            <td class="ms-rteTableHeaderOddCol-0">
                Jan
            </td>
            <td class="ms-rteTableHeaderEvenCol-0">
                Feb
            </td>
            <td class="ms-rteTableHeaderOddCol-0">
                Mar
            </td>
            <td class="ms-rteTableHeaderEvenCol-0">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-0">
            <td class="ms-rteTableEvenCol-0">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-0">
                7
            </td>
            <td class="ms-rteTableOddCol-0">
                5
            </td>
            <td class="ms-rteTableEvenCol-0">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-0">
            <td class="ms-rteTableEvenCol-0">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-0">
                4
            </td>
            <td class="ms-rteTableOddCol-0">
                7
            </td>
            <td class="ms-rteTableEvenCol-0">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-0">
            <td class="ms-rteTableEvenCol-0">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-0">
                7
            </td>
            <td class="ms-rteTableOddCol-0">
                9
            </td>
            <td class="ms-rteTableEvenCol-0">
                24
            </td>
        </tr>
        <tr class="ms-rteTableFooterRow-0">
            <td class="ms-rteTableFooterEvenCol-0">
                &#160;Total
            </td>
            <td class="ms-rteTableFooterOddCol-0">
                &#160;21
            </td>
            <td class="ms-rteTableFooterEvenCol-0">
                18
            </td>
            <td class="ms-rteTableFooterOddCol-0">
                21
            </td>
            <td class="ms-rteTableFooterEvenCol-0">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format 1-C</h2>
    <table class="ms-rteTable-0" style="width: 100%">
        <tr class="ms-rteTableHeaderRow-0">
            <td class="ms-rteTableHeaderFirstCol-0">
                &#160;
            </td>
            <td class="ms-rteTableHeaderOddCol-0">
                Jan
            </td>
            <td class="ms-rteTableHeaderEvenCol-0">
                Feb
            </td>
            <td class="ms-rteTableHeaderOddCol-0">
                Mar
            </td>
            <td class="ms-rteTableHeaderLastCol-0">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-0">
            <td class="ms-rteTableFirstCol-0">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-0">
                7
            </td>
            <td class="ms-rteTableOddCol-0">
                5
            </td>
            <td class="ms-rteTableLastCol-0">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-0">
            <td class="ms-rteTableFirstCol-0">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-0">
                4
            </td>
            <td class="ms-rteTableOddCol-0">
                7
            </td>
            <td class="ms-rteTableLastCol-0">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-0">
            <td class="ms-rteTableFirstCol-0">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-0">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-0">
                7
            </td>
            <td class="ms-rteTableOddCol-0">
                9
            </td>
            <td class="ms-rteTableLastCol-0">
                24
            </td>
        </tr>
        <tr class="ms-rteTableFooterRow-0">
            <td class="ms-rteTableFooterFirstCol-0">
                &#160;Total
            </td>
            <td class="ms-rteTableFooterOddCol-0">
                &#160;21
            </td>
            <td class="ms-rteTableFooterEvenCol-0">
                18
            </td>
            <td class="ms-rteTableFooterOddCol-0">
                21
            </td>
            <td class="ms-rteTableFooterLastCol-0">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format 2-A</h2>
    <table class="ms-rteTable-1" style="width: 100%">
        <tr class="ms-rteTableEvenRow-1">
            <td class="ms-rteTableEvenCol-1">
                &#160;
            </td>
            <td class="ms-rteTableOddCol-1">
                Jan
            </td>
            <td class="ms-rteTableEvenCol-1">
                Feb
            </td>
            <td class="ms-rteTableOddCol-1">
                Mar
            </td>
            <td class="ms-rteTableEvenCol-1">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-1">
            <td class="ms-rteTableEvenCol-1">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-1">
                7
            </td>
            <td class="ms-rteTableOddCol-1">
                5
            </td>
            <td class="ms-rteTableEvenCol-1">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-1">
            <td class="ms-rteTableEvenCol-1">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-1">
                4
            </td>
            <td class="ms-rteTableOddCol-1">
                7
            </td>
            <td class="ms-rteTableEvenCol-1">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-1">
            <td class="ms-rteTableEvenCol-1">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-1">
                7
            </td>
            <td class="ms-rteTableOddCol-1">
                9
            </td>
            <td class="ms-rteTableEvenCol-1">
                24
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-1">
            <td class="ms-rteTableEvenCol-1">
                &#160;Total
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;21
            </td>
            <td class="ms-rteTableEvenCol-1">
                18
            </td>
            <td class="ms-rteTableOddCol-1">
                21
            </td>
            <td class="ms-rteTableEvenCol-1">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format 2-B</h2>
    <table class="ms-rteTable-1" style="width: 100%">
        <tr class="ms-rteTableHeaderRow-1">
            <td class="ms-rteTableHeaderEvenCol-1">
                &#160;
            </td>
            <td class="ms-rteTableHeaderOddCol-1">
                Jan
            </td>
            <td class="ms-rteTableHeaderEvenCol-1">
                Feb
            </td>
            <td class="ms-rteTableHeaderOddCol-1">
                Mar
            </td>
            <td class="ms-rteTableHeaderEvenCol-1">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-1">
            <td class="ms-rteTableEvenCol-1">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-1">
                7
            </td>
            <td class="ms-rteTableOddCol-1">
                5
            </td>
            <td class="ms-rteTableEvenCol-1">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-1">
            <td class="ms-rteTableEvenCol-1">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-1">
                4
            </td>
            <td class="ms-rteTableOddCol-1">
                7
            </td>
            <td class="ms-rteTableEvenCol-1">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-1">
            <td class="ms-rteTableEvenCol-1">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-1">
                7
            </td>
            <td class="ms-rteTableOddCol-1">
                9
            </td>
            <td class="ms-rteTableEvenCol-1">
                24
            </td>
        </tr>
        <tr class="ms-rteTableFooterRow-1">
            <td class="ms-rteTableFooterEvenCol-1">
                &#160;Total
            </td>
            <td class="ms-rteTableFooterOddCol-1">
                &#160;21
            </td>
            <td class="ms-rteTableFooterEvenCol-1">
                18
            </td>
            <td class="ms-rteTableFooterOddCol-1">
                21
            </td>
            <td class="ms-rteTableFooterEvenCol-1">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format 2-C</h2>
    <table class="ms-rteTable-1" style="width: 100%">
        <tr class="ms-rteTableHeaderRow-1">
            <td class="ms-rteTableHeaderFirstCol-1">
                &#160;
            </td>
            <td class="ms-rteTableHeaderOddCol-1">
                Jan
            </td>
            <td class="ms-rteTableHeaderEvenCol-1">
                Feb
            </td>
            <td class="ms-rteTableHeaderOddCol-1">
                Mar
            </td>
            <td class="ms-rteTableHeaderLastCol-1">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-1">
            <td class="ms-rteTableFirstCol-1">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-1">
                7
            </td>
            <td class="ms-rteTableOddCol-1">
                5
            </td>
            <td class="ms-rteTableLastCol-1">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-1">
            <td class="ms-rteTableFirstCol-1">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-1">
                4
            </td>
            <td class="ms-rteTableOddCol-1">
                7
            </td>
            <td class="ms-rteTableLastCol-1">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-1">
            <td class="ms-rteTableFirstCol-1">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-1">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-1">
                7
            </td>
            <td class="ms-rteTableOddCol-1">
                9
            </td>
            <td class="ms-rteTableLastCol-1">
                24
            </td>
        </tr>
        <tr class="ms-rteTableFooterRow-1">
            <td class="ms-rteTableFooterFirstCol-1">
                &#160;Total
            </td>
            <td class="ms-rteTableFooterOddCol-1">
                &#160;21
            </td>
            <td class="ms-rteTableFooterEvenCol-1">
                18
            </td>
            <td class="ms-rteTableFooterOddCol-1">
                21
            </td>
            <td class="ms-rteTableFooterLastCol-1">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format 3-A</h2>
    <table class="ms-rteTable-6" style="width: 100%">
        <tr class="ms-rteTableEvenRow-6">
            <td class="ms-rteTableEvenCol-6">
                &#160;
            </td>
            <td class="ms-rteTableOddCol-6">
                Jan
            </td>
            <td class="ms-rteTableEvenCol-6">
                Feb
            </td>
            <td class="ms-rteTableOddCol-6">
                Mar
            </td>
            <td class="ms-rteTableEvenCol-6">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-6">
            <td class="ms-rteTableEvenCol-6">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-6">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-6">
                7
            </td>
            <td class="ms-rteTableOddCol-6">
                5
            </td>
            <td class="ms-rteTableEvenCol-6">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-6">
            <td class="ms-rteTableEvenCol-6">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-6">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-6">
                4
            </td>
            <td class="ms-rteTableOddCol-6">
                7
            </td>
            <td class="ms-rteTableEvenCol-6">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-6">
            <td class="ms-rteTableEvenCol-6">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-6">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-6">
                7
            </td>
            <td class="ms-rteTableOddCol-6">
                9
            </td>
            <td class="ms-rteTableEvenCol-6">
                24
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-6">
            <td class="ms-rteTableEvenCol-6">
                &#160;Total
            </td>
            <td class="ms-rteTableOddCol-6">
                &#160;21
            </td>
            <td class="ms-rteTableEvenCol-6">
                18
            </td>
            <td class="ms-rteTableOddCol-6">
                21
            </td>
            <td class="ms-rteTableEvenCol-6">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format 3-B</h2>
    <table class="ms-rteTable-6" style="width: 100%">
        <tr class="ms-rteTableHeaderRow-6">
            <td class="ms-rteTableHeaderEvenCol-6">
                &#160;
            </td>
            <td class="ms-rteTableHeaderOddCol-6">
                Jan
            </td>
            <td class="ms-rteTableHeaderEvenCol-6">
                Feb
            </td>
            <td class="ms-rteTableHeaderOddCol-6">
                Mar
            </td>
            <td class="ms-rteTableHeaderEvenCol-6">
                Total
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-6">
            <td class="ms-rteTableEvenCol-6">
                &#160;East
            </td>
            <td class="ms-rteTableOddCol-6">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-6">
                7
            </td>
            <td class="ms-rteTableOddCol-6">
                5
            </td>
            <td class="ms-rteTableEvenCol-6">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-6">
            <td class="ms-rteTableEvenCol-6">
                &#160;West
            </td>
            <td class="ms-rteTableOddCol-6">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-6">
                4
            </td>
            <td class="ms-rteTableOddCol-6">
                7
            </td>
            <td class="ms-rteTableEvenCol-6">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-6">
            <td class="ms-rteTableEvenCol-6">
                &#160;South
            </td>
            <td class="ms-rteTableOddCol-6">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-6">
                7
            </td>
            <td class="ms-rteTableOddCol-6">
                9
            </td>
            <td class="ms-rteTableEvenCol-6">
                24
            </td>
        </tr>
        <tr class="ms-rteTableFooterRow-6">
            <td class="ms-rteTableFooterEvenCol-6">
                &#160;Total
            </td>
            <td class="ms-rteTableFooterOddCol-6">
                &#160;21
            </td>
            <td class="ms-rteTableFooterEvenCol-6">
                18
            </td>
            <td class="ms-rteTableFooterOddCol-6">
                21
            </td>
            <td class="ms-rteTableFooterEvenCol-6">
                60
            </td>
        </tr>
    </table>
    <h2>
        Table Format 3-C</h2>
    <table class="ms-rteTable-6" style="width: 100%">
        <tr class="ms-rteTableHeaderRow-6">
            <th class="ms-rteTableHeaderFirstCol-6">
            </th>
            <th class="ms-rteTableHeaderOddCol-6">
                Jan
            </th>
            <th class="ms-rteTableHeaderEvenCol-6">
                Feb
            </th>
            <th class="ms-rteTableHeaderOddCol-6">
                Mar
            </th>
            <th class="ms-rteTableHeaderLastCol-6">
                Total
            </th>
        </tr>
        <tr class="ms-rteTableOddRow-6">
            <th class="ms-rteTableFirstCol-6">
                &#160;East
            </th>
            <td class="ms-rteTableOddCol-6">
                &#160;7
            </td>
            <td class="ms-rteTableEvenCol-6">
                7
            </td>
            <td class="ms-rteTableOddCol-6">
                5
            </td>
            <td class="ms-rteTableLastCol-6">
                19
            </td>
        </tr>
        <tr class="ms-rteTableEvenRow-6">
            <th class="ms-rteTableFirstCol-6">
                &#160;West
            </th>
            <td class="ms-rteTableOddCol-6">
                &#160;6
            </td>
            <td class="ms-rteTableEvenCol-6">
                4
            </td>
            <td class="ms-rteTableOddCol-6">
                7
            </td>
            <td class="ms-rteTableLastCol-6">
                17
            </td>
        </tr>
        <tr class="ms-rteTableOddRow-6">
            <th class="ms-rteTableFirstCol-6">
                &#160;South
            </th>
            <td class="ms-rteTableOddCol-6">
                &#160;8
            </td>
            <td class="ms-rteTableEvenCol-6">
                7
            </td>
            <td class="ms-rteTableOddCol-6">
                9
            </td>
            <td class="ms-rteTableLastCol-6">
                24
            </td>
        </tr>
        <tr class="ms-rteTableFooterRow-6">
            <th class="ms-rteTableFooterFirstCol-6">
                &#160;Total
            </th>
            <td class="ms-rteTableFooterOddCol-6">
                &#160;21
            </td>
            <td class="ms-rteTableFooterEvenCol-6">
                18
            </td>
            <td class="ms-rteTableFooterOddCol-6">
                21
            </td>
            <td class="ms-rteTableFooterLastCol-6">
                60
            </td>
        </tr>
    </table>
</body>
</html>
```

I typically use a "TestConsole" utility to programmatically create a Style Guide  page in a SharePoint site for development and testing purposes. This makes it very  easy to recreate the Style Guide page in new environments or after rebuilding a  SharePoint Web application in DEV and TEST.

