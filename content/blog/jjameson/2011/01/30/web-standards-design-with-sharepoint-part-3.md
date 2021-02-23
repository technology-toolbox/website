---
title: "Web Standards Design with SharePoint, Part 3"
date: 2011-01-30T20:29:00-07:00
excerpt: "Last week I received the following comment on a blog post I wrote last year regarding Web standards design with Microsoft Office SharePoint Server (MOSS) 2007: 
 
 \"The Media Guy\" 
 
 Great article.. was very helpful. I used 960.gs for my master page..."
aliases: ["/blog/jjameson/archive/2011/01/30/web-standards-design-with-sharepoint-part-3.aspx"]
draft: true
categories: ["SharePoint", "Development"]
tags: ["MOSS 2007", "Web Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/01/30/web-standards-design-with-sharepoint-part-3.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/01/30/web-standards-design-with-sharepoint-part-3.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

Last week I received the following comment on [a blog post I wrote last year](/blog/jjameson/2010/01/30/web-standards-design-with-moss-2007-part-1) regarding Web standards design with Microsoft         Office SharePoint Server (MOSS) 2007:

1. <cite>"The Media Guy"</cite>
   {{< blockquote "font-italic" >}}

Great article.. was very helpful. I used 960.gs for my master page as well and all                     is good. I am now creating a 3 column page layout . I need a grid\_3 (left), grid\_6                     (middle), and a grid\_3 (right). I would like these all to be blank web part zones.                     I started off with the "Blank web part page" as a template but it is using nested                     Tables and really hard to look at. Do you have any advice for a starting a page                     layout based on 960.gs?

{{< /blockquote >}}

Rather than trying to explain to "The Media Guy" how to create a page layout based         on the [960 Grid System](http://960.gs/), this weekend I revisited the         sample I started creating last year as a follow-up to my original post. However,         to keep things as simple as possible -- while still demonstrating valuable, "real         world" concepts -- I updated the master page and page layouts for "Fabrikam Technologies"         based on the out-of-the-box Collaboration Portal site definition (which has a fair         amount of content populated by default). [Originally, my Web standards demo site         was based on the Internet site for the fictitious "Adventure Works" bicycle company.]

If you've worked with MOSS 2007 at all, then you are undoubtedly familiar with the         following:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/DefaultMaster_DefaultPageLayout-600x284.png"
alt="Home page for \"Collaboration Portal\" (a.k.a. a \"starter site hierarchy for an intranet divisional portal\")"
class="screenshot"
height="284"
width="600"
title="Figure 1: Home page for \"Collaboration Portal\" (a.k.a. a \"starter site hierarchy for an intranet divisional portal\")" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/DefaultMaster_DefaultPageLayout-1064x504.png)

Let's suppose that Fabrikam Technologies wants to deploy SharePoint for their corporate         intranet, but they want to brand the site with their corporate logo, font, and color         scheme. Furthermore, the folks at Fabrikam really want us -- at least as much as         possible -- to use all the goodness of Web standards design (in other words, CSS-based         layout instead of that nasty ol' table-based layout from years ago).

As I described in my original post -- and also demonstrated in [part 2 of this series](/blog/jjameson/2010/12/02/web-standards-design-with-sharepoint-part-2) (although not using the 960 Grid System in that example)         -- you can certainly design SharePoint sites using Web standards (even though table-based         layout is rampant in the out-of-the-box SharePoint controls, master pages, and page         layouts).

Here's a screenshot showing the home page using the custom Fabrikam master page         and page layout:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/FabrikamDefaultMaster-DefaultLayout-600x299.png"
alt="Home page with custom master page and page layout"
class="screenshot"
height="299"
width="600"
title="Figure 2: Home page with custom master page and page layout" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/FabrikamDefaultMaster-DefaultLayout-1062x530.png)

Note that the actual page content is identical in the two figures above -- I've         simply "re-skinned" the site to look substantially different from the default SharePoint         look-and-feel. Also note that there are two fundamental problems with the out-of-the-box         page content.

First, in the **Page Content** field, the default content is far from         "semantic HTML" and contains numerous nested tables (to achieve the two-column layout).         This is easily seen in the following screenshot, in which I've disabled the linked         style sheets:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/FabrikamDefaultMaster-DefaultLayout2-600x434.png"
alt="Home page with custom master page and page layout (linked CSS files disabled)"
class="screenshot"
height="434"
width="600"
title="Figure 3: Home page with custom master page and page layout (linked CSS files disabled)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/FabrikamDefaultMaster-DefaultLayout2-1045x756.png)

Second, the default page layout does not expose either of the Summary Links fields         defined on the **Welcome Page** content type. Consequently, a Web Part         (i.e. "News") is used to render the links at the bottom of the page.

> **Note**
>
> You should always try to use the Summary Links fields (from the underlying content type) instead of Summary Links Web Parts whenever possible -- since this provides the ability to track the changes to the links over time. (In other words, changes to Web Part properties are not shown in the **Version History** for a page, whereas changes to a Summary Links field are shown.)

To mitigate these issues, I chose to tweak the default content for the home page         (to replace the **Page Content** with semantic HTML -- while still         preserving the two-column layout within the field -- and to replace the "News" Web         Part with corresponding Summary Links).

The following figure shows the end result (note that I overlayed the grid using         the [960 Gridder](http://gridder.andreehansson.se/) for illustrative         purposes):

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/FabrikamDefaultMaster-CustomLayout-600x361.png"
alt="Home page with custom master page and page layout (semantic HTML and Summary Links field instead of Web Part)"
class="screenshot"
height="361"
width="600"
title="Figure 4: Home page with custom master page and page layout (semantic HTML and Summary Links field instead of Web Part)" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/FabrikamDefaultMaster-CustomLayout-1045x628.png)

Here are the instructions to deploy the Fabrikam Demo sample to your own SharePoint         environment. First, download the attachment and unzip the files. Then you simply         need to run a few commands from an Administrator command prompt. [As I've warned         in the past, you should obviously inspect my scripts before blindly running them         with administrative privileges, but honestly, there's nothing malicious in there.]

To deploy the Fabrikam Demo to SharePoint:

1. Click **Start**, point to **All Programs**, point to **Accessories**, and right-click **Command Prompt**, and then
   click **Run as administrator**.

2. At the command prompt, type the following command to set the enviroment variable
   corresponding to a local (developer) environment:
   
   ```
   set FABRIKAM_INTRANET_URL=http://fabweb-local
   ```

> **Note**
>
> While you don't have to use this URL, it is recommended for developer environments because it causes the deployment scripts to bypass the SharePoint timer infrastructure when deploying and retracting the solution.
> 3. Set environment variables to specify the credentials to use for the Fabrikam application
> pool:

    ```
    set FABRIKAM_APP_POOL_IDENTITY=%USERDOMAIN%\svc-web-fabrikam-dev
    set FABRIKAM_APP_POOL_PASSWORD={password}
    ```

> **Important**
>
> Be sure to specify a valid local or domain user.
> 4. Change to the folder containing the deployment scripts:

{{< console-block-start >}}

cd Demo\Dev\SharePointDevelopment\Source\DeploymentFiles\Scripts

{{< console-block-end >}}
5. Type the following command:

    ```
    "Create Web Applications.cmd"
    ```

6. Wait for the new Web application and corresponding site collection to be created,
   and then type the following command:
   
   ```
   "Add Solutions.cmd"
   ```

7. Wait for the solution to be added and then type the following command:
   
   ```
   "Deploy Solutions.cmd"
   ```

8. Wait for the solution to be deployed and then type the following command:
   
   ```
   "Activate Features.cmd"
   ```

9. Wait for the feature activations to complete, and then minimize or close the command
   prompt.

That's it! (During feature activation, I automatically set the master page on the         site, change the page layout of the home page, and update the corresponding content.)

You are now ready to browse to the Fabrikam intranet site and review the master         page and page layouts leveraging the 960 Grid System.

> **Tip**
>
> Once you have the site running in your environment, click **Sample Link 1** and **Sample Link 2** on the home page to view a couple of test pages that are created to demonstrate other custom page layouts.

