---
title: "Migrating a blog from Telligent to Subtext using Web scraping, the Html Agility Pack, BlogML, and more (a.k.a. Building TechnologyToolbox.com, part 6)"
date: 2011-11-12T23:51:46-07:00
lastmod: 2011-11-12T23:52:07-07:00
excerpt: "While I don't expect many people will need to migrate blog posts from Telligent to Subtext, I do believe it is valuable to provide a walkthrough of how I typically approach a \"content migration scenario\" -- since I frequently encounter this kind of requirement when working with enterprise customers..."
aliases: ["/blog/jjameson/archive/2011/11/12/building-technologytoolbox-com-part-6.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["My System", "Subtext", "Web Development"]
---

While I don't expect many people will need to migrate blog posts from Telligent to Subtext, I do believe it is valuable to provide a walkthrough of how I typically approach a "content migration scenario" -- since I frequently encounter this kind of requirement when working with enterprise customers.

For example, a big part of the [Agilent Technologies](http://chem.agilent.com/) project I worked on a few years ago involved migrating their legacy Internet site from a proprietary ASP application to Microsoft Office SharePoint Server 2007. Consequently, we ended up creating a number of custom migration utilities to export content from the old system and subsequently import it into SharePoint.

When setting up my new blog on TechnologyToolbox.com, I knew that I wanted to copy all of the posts from [my old MSDN blog](http://blogs.msdn.com/b/jjameson/), including the corresponding tags and comments. However, as you can imagine, when I decided to leave Microsoft I obviously couldn't ask for a backup of the Telligent database in order to extract my blog post content and corresponding comments.

Therefore I knew I would have to do some "[Web scraping](http://en.wikipedia.org/wiki/Web_scraping)" if I wanted to ensure the preservation of all the blog posts I spent countless hours creating in years past.

One of the great features in Subtext is the ability to import blog posts using [BlogML](http://en.wikipedia.org/wiki/BlogML) (including categories, tags, and comments). Therefore I knew the migration effort would primarily involve exporting content from the old platform (Telligent) into the structure (BlogML) supported by the new platform (Subtext). Conceptually, this is very similar to how I used the "PRIME" API a few years ago on the Agilent project to import content into SharePoint.

> **Note**
>
> If your new platform supports some kind of import functionality, then much of the "heavy lifting" involved in migrating content has already been done. You just need to figure out how to export content from the source system into the "schema" expected by the import feature of the destination system.

Here is the overall approach I used to migrate my content from Telligent to Subtext:

1. Download the monthly summary pages to a temporary cache (e.g. [http://blogs.msdn.com/b/jjameson/archive/2007/03.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/03.aspx) → C:\NotBackedUp\Temp\MSDN-blog\Post Summaries\Monthly Summary 2007-03.html).
2. Create a new BlogML "document" (using [BlogML .NET](http://blogml.codeplex.com/)).
3. Parse each summary page to get the posts for a particular month and add them to the BlogML document.
4. Download each blog post to a temporary cache and then parse the HTML (using the [Html Agility Pack](http://htmlagilitypack.codeplex.com/)) to extract the "main content" of each blog post as well as the tags.
5. Export the comments for each post and add them to the BlogML document.
6. Save the BlogML document to an XML file.
7. Using the Subtext admin interface, import the BlogML file to create the list of categories and blog posts.

### Step 1: Download the monthly summary pages

I started by creating a C# console application and adding a small amount of code to download the monthly summary pages from my MSDN blog to a temporary folder.

```
        static void Main(string[] args)
        {
            Uri oldBlogBaseUrl = new Uri("http://blogs.msdn.com/b/jjameson/");

            const string summaryFolder =
                @"C:\NotBackedUp\Temp\MSDN-blog\Post Summaries";

            ExportSummaryPages(oldBlogBaseUrl, summaryFolder);
        }
```

Note that Telligent allows you to view the list of posts for a particular month by browsing to a URL of the form "archive/{year}/{month}.aspx" (e.g. [http://blogs.msdn.com/b/jjameson/archive/2007/03.aspx](http://blogs.msdn.com/b/jjameson/archive/2007/03.aspx)). [Subtext also provides similar functionality (as do most other blogging solutions out there, I imagine). However, unlike Telligent, in Subtext the <var>{month}</var> portion of the URL must be specified using the two-digit format.]

Since I authored my first blog post back in 2007, I created a couple of `for` loops to iterate over all months in the last 4 years and download the corresponding Web page to an offline file.

```
        private static void ExportSummaryPages(
            Uri oldBlogBaseUrl,
            string summaryFolder)
        {
            Debug.Assert(oldBlogBaseUrl.AbsoluteUri.EndsWith("/") == true);

            int yearFirstBlogPostCreated = 2007;

            for (int year = yearFirstBlogPostCreated; year <= DateTime.Now.Year;
                year++)
            {
                for (int month = 1; month < 13; month++)
                {
                    string summaryPageRelativeUrl = string.Format(
                        "archive/{1}/{2:D2}.aspx",
                        oldBlogBaseUrl,
                        year,
                        month);

                    Uri summaryPageUrl = new Uri(
                        oldBlogBaseUrl,
                        summaryPageRelativeUrl);

                    string summaryPageFile = string.Format(
                        @"{0}\Monthly Summary {1}-{2:D2}.html",
                        summaryFolder,
                        year,
                        month);

                    EnsureOfflineFile(summaryPageUrl, summaryPageFile);

                    // HACK: 28 blog posts created in October 2009 (but summary
                    // pages only show 25 at a time)
                    if (year == 2009 && month == 10)
                    {
                        summaryPageUrl = new Uri(
                            oldBlogBaseUrl,
                            "archive/2009/10.aspx?PageIndex=2");

                        summaryPageFile =
                            summaryFolder
                            + @"\Monthly Summary 2009-10 Page 2.html";

                        EnsureOfflineFile(summaryPageUrl, summaryPageFile);
                    }
                }
            }
        }
```

The **EnsureOfflineFile** method simply checks to see if the specified file exists and, if not, uses the **[WebClient](http://msdn.microsoft.com/en-us/library/system.net.webclient.aspx)** class in the .NET Framework to download it (but first ensuring the specified destination folder exists -- since the [**DownloadFile**](http://msdn.microsoft.com/en-us/library/ms144194.aspx) method won't automatically create any necessary folders):

```
        private static void EnsureOfflineFile(
            Uri url,
            string offlineFilename)
        {
            if (File.Exists(offlineFilename) == false)
            {
                Console.WriteLine("Downloading {0}...", url);

                string archiveFolder = Path.GetDirectoryName(offlineFilename);

                if (Directory.Exists(archiveFolder) == false)
                {
                    Directory.CreateDirectory(archiveFolder);
                }

                WebClient client = new WebClient();
                client.DownloadFile(url, offlineFilename);
            }
        }
```

> **Important**
>
> When migrating content from one system to another, I strive to minimize the load on the source system -- in this case, by downloading the files one time from the MSDN blog site. While I could certainly download these pages each time the program runs, this would put an unnecessary load on the MSDN site (and also significantly slow down the process of developing the content migration utility).
>
> Note that this is just one way of minimizing the impact of migrating content. Another common approach that I've used in the past is to initially run the content migration against the DEV environment and later on against the TEST environment for validation and verification purposes. The final content migration is subsequently performed against the PROD environment one time only (ideally).

### Step 2: Create a new BlogML document

While I could have chosen to "handcraft" a BlogML file using something like **XmlTextWriter** or **XmlDocument**, during my initial research for migrating my blog I discovered [BlogML .NET](http://blogml.codeplex.com/), which makes it very easy to create the corresponding XML file with substantially less coding on my part.

```
        static void Main(string[] args)
        {
            ...

            Uri newBlogBaseUrl = new Uri(
                "https://www.technologytoolbox.com/blog/jjameson/");

            ...

            BlogMLBlog blog = new BlogMLBlog();
            blog.Title = "Random Musings of Jeremy Jameson";
            blog.RootUrl = newBlogBaseUrl.AbsolutePath;
            blog.DateCreated = new DateTime(2005, 5, 1);

            BlogMLAuthor author = new BlogMLAuthor();
            author.DateCreated = blog.DateCreated;
            author.Email = "jjameson@technologytoolbox...";
            author.Approved = true;
            author.DateModified = DateTime.Now;
            author.ID = "0";
            author.Title = "Jeremy Jameson";

            blog.Authors.Add(author);

            Pair<string, string> item = new Pair<string, string>(
                "CommentModeration",
                "Disabled");

            blog.ExtendedProperties.Add(item);
        }
```

Note that by the time I had written this code, I had already exported a sample BlogML file from Subtext and observed the following in the XML file:

```
  <extended-properties>
    <property name="CommentModeration" value="Disabled" />
  </extended-properties>
```

To ensure this appeared in the file generated from my migration utility, I added code to explicitly specify this extended property.

### Step 3: Load the summary for each post into the BlogML document

The next step is to iterate through the offline archive files for each month and add a limited amount of information for each post into the **BlogMLBlog** instance.

```
        static void Main(string[] args)
        {
            ...

            const string summaryFolder =
                @"C:\NotBackedUp\Temp\MSDN-blog\Post Summaries";

            ...

            BlogMLBlog blog = new BlogMLBlog();

            ...

            ExportPostSummaries(blog, summaryFolder);
        }
```

**ExportPostSummaries** enumerates each HTML file in the "summary folder" (e.g. C:\NotBackedUp\Temp\MSDN-blog\Post Summaries\Monthly Summary 2007-03.html), looks for a specific string (`"No blog posts have yet been created"`) to see if, in fact, there are any posts for the corresponding month, and then parses the HTML using an instance of the **HtmlDocument** class from the Html Agility Pack.

```
        private static void ExportPostSummaries(
            BlogMLBlog blog,
            string summaryFolder)
        {
            string[] filenames = Directory.GetFiles(summaryFolder);

            foreach (string filename in filenames)
            {
                string summaryPageContent = File.ReadAllText(filename);

                if (summaryPageContent.Contains(
                    "No blog posts have yet been created") == true)
                {
                    continue;
                }

                HtmlWeb htmlWeb = new HtmlWeb();

                HtmlDocument document = htmlWeb.Load(filename);

                HtmlNodeCollection abbreviatedPosts =
                    document.DocumentNode.SelectNodes(
                        "//*[@class='abbreviated-post']");

                foreach (HtmlNode abbreviatedPost in abbreviatedPosts)
                {
                    BlogMLPost post = new BlogMLPost();

                    HtmlNode postHeading = abbreviatedPost.SelectSingleNode(
                        "h4");

                    Debug.Assert(postHeading != null);

                    post.Title = HttpUtility.HtmlDecode(postHeading.InnerText);

                    HtmlNode postLink = postHeading.SelectSingleNode("a");
                    Debug.Assert(postHeading != null);

                    post.PostUrl = postLink.Attributes["href"].Value;

                    HtmlNode postSummary = abbreviatedPost.SelectSingleNode(
                        "div[@class='post-summary']");

                    Debug.Assert(postSummary != null);

                    post.Excerpt = BlogMLContent.Create(
                        postSummary.InnerText,
                        ContentTypes.Text);

                    post.HasExcerpt = true;

                    post.Authors.Add(blog.Authors[0].ID);

                    blog.Posts.Add(post);
                }
            }
        }
```

Even if you haven't used the Html Agility Pack before (which is something I highly recommend if you ever encounter the need to parse HTML), it should be relatively easy to understand the parsing logic in the **ExportPostSummaries** method. Here is a sample HTML fragment that shows the "interesting" elements that are used to extract the post title and excerpt (i.e. summary) and add these to the BlogML document.

```
<div class="abbreviated-post">
    <div class="post-application">
        ...
    </div>
    <h4 class="post-name">
        <a class="internal-link view-post"
            href="/b/jjameson/archive/2007/03/23/sql-server-authentication-modes.aspx">
            <span></span>SQL Server Authentication Modes</a></h4>
    <div class="post-date">
        ...
    </div>
    <div class="post-author">
        ...
    </div>
    <div class="post-attributes">
        ...
    </div>
    <div class="post-summary">
        A fellow consultant here in Denver sent out a message yesterday
        inquiring about formal recommendations regarding the use of Windows
        Authentication vs. SQL Authentication. It seems that quite a few
        customers out there ponder these choices for a variety...</div>
</div>
```

> **Tip**
>
> Parsing with the Html Agility Pack works very well when the source HTML is dynamically generated using a template mechanism (like ASP.NET or PHP) because the structure is very consistent. Handcrafted HTML, on the other hand, would likely prove very difficult -- or downright impossible -- to parse in a reliable manner.

### Step 4: Download each blog post and parse the HTML

At this point, the instance of **BlogMLBlog** contains about 300 posts, but only the title and excerpt of each post. The next step is to populate the full content for each post as well as set some additional metadata (e.g. the date each post was published). This functionality is implemented in the **ExportPosts** method.

```
        static void Main(string[] args)
        {
            Uri oldBlogBaseUrl = new Uri("http://blogs.msdn.com/b/jjameson/");
            Uri newBlogBaseUrl = new Uri(
                "https://www.technologytoolbox.com/blog/jjameson/");

            ...

            const string postsFolder =
                @"C:\NotBackedUp\Temp\MSDN-blog";

            ...

            BlogMLBlog blog = new BlogMLBlog();

            ...

            ExportPosts(blog, postsFolder, oldBlogBaseUrl, newBlogBaseUrl);
        }
```

Having previously populated the BlogML document, it is now simply a matter of iterating the collection of posts and processing each one individually. The first step is to download an offline copy of the post using the **EnsureOfflineFile** method shown above.

```
        private static void ExportPosts(
            BlogMLBlog blog,
            string postsFolder,
            Uri oldBlogBaseUrl,
            Uri newBlogBaseUrl)
        {
            foreach (BlogMLPost post in blog.Posts)
            {
                Uri originalPostUrl = GetOriginalPostUrl(
                    post,
                    oldBlogBaseUrl);

                string relativePath = originalPostUrl.AbsolutePath.Substring(
                    oldBlogBaseUrl.AbsolutePath.Length);

                relativePath = relativePath.Replace('/', '\\');

                string offlinePostFilename = Path.Combine(
                    postsFolder,
                    relativePath);

                Debug.Assert(offlinePostFilename.EndsWith(".aspx") == true);

                offlinePostFilename = Path.ChangeExtension(
                    offlinePostFilename,
                    ".html");

                EnsureOfflineFile(originalPostUrl, offlinePostFilename);

                FillPostDetail(
                    blog,
                    post,
                    offlinePostFilename,
                    oldBlogBaseUrl,
                    newBlogBaseUrl);

                post.PostUrl = originalPostUrl.AbsoluteUri.Replace(
                    oldBlogBaseUrl.AbsoluteUri,
                    newBlogBaseUrl.AbsoluteUri);
            }
        }
```

**FillPostDetail** is where things start to get more interesting...

```
        private static void FillPostDetail(
            BlogMLBlog blog,
            BlogMLPost post,
            string postFile,
            Uri oldBlogBaseUrl,
            Uri newBlogBaseUrl)
        {
            string html = File.ReadAllText(postFile);

            HtmlDocument document = new HtmlDocument();
            document.OptionWriteEmptyNodes = true;
            document.OptionOutputAsXml = true;
            document.LoadHtml(html);

            HtmlNode postDate = document.DocumentNode.SelectSingleNode(
                    "//div[@class='post-date']");

            Debug.Assert(postDate != null);
            post.DateCreated = DateTime.Parse(postDate.InnerText);
            post.DateModified = post.DateCreated;
            post.Approved = true;
            post.PostName = Path.GetFileNameWithoutExtension(postFile);

            Uri originalPostUrl = GetOriginalPostUrl(post, oldBlogBaseUrl);

            StringBuilder buffer = new StringBuilder();
            buffer.Append("<blockquote class='note original-post'>");
            buffer.Append("<div class='noteTitle'><strong>Note</strong></div>");
            buffer.Append("<div>This post originally appeared on my MSDN"
                + " blog:<br /><br />");

            buffer.Append("<div class='reference'>");
            buffer.AppendFormat("<div class='referenceLink'><a href='{0}'>{0}</a></div>",
                originalPostUrl);

            buffer.Append("</div>");
            buffer.Append("<p>Since <a"
+ " href='/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx'>"
                + "I no longer work for Microsoft</a>, I have copied it here in"
                + " case that blog ever goes away.</p>");

            buffer.Append("</div>");
            buffer.Append("</blockquote>");

            HtmlNode postContent = document.DocumentNode.SelectSingleNode(
                    "//div[@class='post-content user-defined-markup']");

            Debug.Assert(postContent != null);

            string newPostContent = TransformOriginalPostContent(
                postContent,
                originalPostUrl,
                newBlogBaseUrl);

            buffer.Append(newPostContent);

            AppendBlogPostTags(
                document,
                buffer);

            newPostContent = buffer.ToString();

            post.Content = BlogMLContent.Create(
                newPostContent,
                ContentTypes.Html);

            FillPostCategories(blog, post, document);
        }
```

Since I want to ensure my blog posts are XHTML-compliant, I set a couple of options on the corresponding **HtmlDocument** used for parsing. I then parse the date the post was published and use it for both the "created" date as well as the "modified" date. This is a slight deviation from reality since I updated some posts after they were originally published as well as the fact that I didn't always publish posts on the same day I originally created them -- but these are insignificant issues in this scenario. [On other migration projects I have worked on, preserving timestamps was deemed an essential requirement. Be sure to weigh the development effort involved in trying to preserve the "absolute truth" with the corresponding value to the business.]

Next I add a note to each migrated post (indicating that it originally appeared on my MSDN blog and then proceed to parsing the main "body" of the post (specified in the `<div class="post-content user-defined-markup">` element).

#### Transforming blog post content

As its name implies, the **TransformOriginalPostContent** method "massages" the content of each post in order to:

- Remove "junk" added by the Telligent WYSIWYG HTML editor that I don't want, such as `mce_href` attributes (especially since these attributes break XHTML-compliance)
- Update links within a blog post to other posts on my blog (since I obviously want to keep visitors on my new site rather than having them get redirected over to my old MSDN blog)
- Ensure the post content is well-formed (and thus at least has a "fighting chance" at being XHTML-compliant)
- Replace miscellaneous "special" characters in the HTML that typically result from copying/pasting from Microsoft Word (e.g. curly quotes) -- since I frequently copy text from documents I've written when creating blog posts

```
        private static string TransformOriginalPostContent(
            HtmlNode postContent,
            Uri originalPostUrl,
            Uri newBlogBaseUrl)
        {
            RemoveUnwantedAttributes(postContent, "mce_href");
            RemoveUnwantedAttributes(postContent, "mce_keep");
            RemoveUnwantedAttributes(postContent, "mce_src");

            TranslateLinksToOtherBlogPosts(postContent, newBlogBaseUrl);

            HtmlNode lastElement =
                postContent.ChildNodes[postContent.ChildNodes.Count - 1];

            Debug.Assert(lastElement.Name == "div");
            Debug.Assert(lastElement.Attributes["style"].Value == "clear:both;");
            postContent.RemoveChild(lastElement);

            string htmlErrors;

            string newPostContent = HtmlCleaner.CleanFragment(
                postContent.InnerHtml,
                out htmlErrors);

            if (string.IsNullOrEmpty(htmlErrors) == false)
            {
                Console.WriteLine(
                    "One or more errors detected in HTML for post ({0}):"
                        + Environment.NewLine
                        + "{1}",
                    originalPostUrl,
                    htmlErrors);
            }

            newPostContent = newPostContent.Replace("&amp;ndash;", "--");
            newPostContent = newPostContent.Replace("&amp;ldquo;", "\"");
            newPostContent = newPostContent.Replace("&amp;rdquo;", "\"");
            newPostContent = newPostContent.Replace("&amp;rsquo;", "'");

            return newPostContent;
        }
```

Removing unwanted attributes in the HTML turns out to be rather trivial, thanks to the powerful XPath query capabilities of the Html Agility Pack:

```
        private static void RemoveUnwantedAttributes(
            HtmlNode postContent,
            string attributeName)
        {
            HtmlNodeCollection nodes = postContent.SelectNodes(
                "//@" + attributeName);

            if (nodes == null)
            {
                return;
            }

            foreach (HtmlNode node in nodes)
            {
                node.Attributes.Remove(attributeName);
            }
        }
```

Link translation is also relatively easy thanks to the **HtmlNode.Descendants** method and the fact that the URLs for each post on my new blog are very similar to the old URLs:

```
        private static void TranslateLinksToOtherBlogPosts(
            HtmlNode postContent,
            Uri newBlogBaseUrl)
        {
            Debug.Assert(newBlogBaseUrl.AbsoluteUri.EndsWith("/") == true);

            IEnumerable<HtmlNode> nodes = postContent.Descendants("a");

            foreach (HtmlNode node in nodes)
            {
                if (node.Attributes.Contains("href") == false)
                {
                    continue;
                }
                else if (node.Attributes["href"].Value.StartsWith("#") == true)
                {
                    continue;
                }

                Uri href = new Uri(node.Attributes["href"].Value);

                if (href.AbsoluteUri.StartsWith(
                    "http://blogs.msdn.com/jjameson/",
                    StringComparison.OrdinalIgnoreCase) == true)
                {
                    string relativeUrl = href.AbsolutePath.Substring(
                        "/jjameson/".Length);

                    Uri newHref = new Uri(
                        newBlogBaseUrl,
                        relativeUrl);

                    node.Attributes["href"].Value = newHref.AbsolutePath;
                }
                else if (href.AbsoluteUri.StartsWith(
                    "http://blogs.msdn.com/b/jjameson/",
                    StringComparison.OrdinalIgnoreCase) == true)
                {
                    string relativeUrl = href.AbsolutePath.Substring(
                        "/b/jjameson/".Length);

                    Uri newHref = new Uri(
                        newBlogBaseUrl,
                        relativeUrl);

                    node.Attributes["href"].Value = newHref.AbsolutePath;
                }
            }
        }
```

On the Agilent project, for example, link translation was much more involved. We ended up using a database to provide "lookup" functionality to translate URLs from the legacy system to the new URLs within SharePoint.

Note that I originally tried using only the Html Agility Pack to ensure the post content is well-formed. However, I found some instances in my blog posts where the **HtmlDocument** class did not fix mismatched tags.

Rather than attempting to debug somebody else's code, I decided to "punt" and just use my **HtmlCleaner** class instead -- since I've used this extensively in the past. Note that **HtmlCleaner** is really just a thin wrapper around **[SgmlReader](http://archive.msdn.microsoft.com/SgmlReader):**

```
    /// <summary>
    /// Utility class for ensuring HTML is well-formed.
    /// </summary>
    internal static class HtmlCleaner
    {
        /// <summary>
        /// Ensures the specified HTML is well-formed.
        /// </summary>
        /// <param name="htmlInput">The content to convert to well-formed HTML.
        /// </param>
        /// <param name="errors">Errors detected during the "cleanup" of the
        /// HTML input.</param>
        /// <returns>A string containing the well-formed HTML based on the
        /// specified input.</returns>
        public static string Clean(
            string htmlInput,
            out string errors)
        {
            errors = null;

            if (string.IsNullOrEmpty(htmlInput) == true)
            {
                return htmlInput;
            }

            using (StringReader sr = new StringReader(htmlInput))
            {
                using (StringWriter errorWriter = new StringWriter(
                    CultureInfo.CurrentCulture))
                {
                    SgmlReader reader = new SgmlReader();
                    reader.DocType = "HTML";
                    reader.CaseFolding = CaseFolding.ToLower;
                    reader.InputStream = sr;
                    reader.ErrorLog = errorWriter;

                    using (StringWriter sw = new StringWriter(
                        CultureInfo.CurrentCulture))
                    {
                        using (XmlTextWriter w = new XmlTextWriter(sw))
                        {
                            reader.Read();
                            while (reader.EOF == false)
                            {
                                w.WriteNode(reader, true);
                            }
                        }

                        errorWriter.Flush();
                        errors = errorWriter.ToString();

                        string cleanedHtml = sw.ToString();

                        return cleanedHtml;
                    }
                }
            }
        }

        /// <summary>
        /// Ensures the specified HTML fragment is well-formed.
        /// </summary>
        /// <param name="htmlFragment">The content to convert to well-formed
        /// HTML.</param>
        /// <param name="errors">Errors detected during the "cleanup" of the
        /// HTML input.</param>
        /// <returns>A string containing the well-formed HTML based on the
        /// specified input.</returns>
        public static string CleanFragment(
            string htmlFragment,
            out string errors)
        {
            errors = null;

            if (string.IsNullOrEmpty(htmlFragment) == true)
            {
                return htmlFragment;
            }

            string cleanedHtml = Clean(htmlFragment, out errors);

            Debug.Assert(string.IsNullOrEmpty(cleanedHtml) == false);
            Debug.Assert(cleanedHtml.StartsWith(
                "<html>",
                StringComparison.OrdinalIgnoreCase) == true);

            Debug.Assert(cleanedHtml.EndsWith(
                "</html>",
                StringComparison.OrdinalIgnoreCase) == true);

            cleanedHtml = cleanedHtml.Substring(
                "<html>".Length,
                cleanedHtml.Length - "<html>".Length - "</html>".Length);

            return cleanedHtml;
        }
    }
```

> **Important**
>
> This is probably a good place to stop and point out an essential concept when writing migration tools like this -- as well as similar kinds of "throw away" code.
>
> Keep in mind the ultimate goal of this utility is to run it *one time* (after all of the development and testing effort is completed, of course). In other words, when I'm writing utilities like this, I try not to focus on writing highly maintainable code. Hence you don't see any significant error handling, parameter validation, boundary checking, etc.
>
> The goal is to complete the code as quickly as possible -- ensuring that it is "good enough" to do its job but doesn't necessarily demonstrate best coding practices. For example, I don't bother enabling *all* code analysis rules on "throw away" code like this (which is typically one of the first things I do after creating a new project in Visual Studio). Mind you, I definitely add these kinds of utilities to source control, but that's primarily as a benefit during development so I can easily rollback unwanted changes (and occasionally to go back at some point in the future and review the code for reference purposes).
>
> If this utility was something I expected to be maintained going forward, I would have spent more time trying to figure out why I couldn't get the Html Agility Pack to fix the malformed HTML.

#### Migrating tags and categories

I've mentioned before how Subtext supports both *tags* and *categories* for blog posts. At first, I didn't really see the need for having two separate taxonomies (especially since my MSDN blog only supported tags), but the more I thought about it, the more I started to like the distinction.

In the end, I settled on about a half dozen categories:

- Development
- Infrastructure
- My System
- Personal
- SharePoint

I debated for a little while about whether or not to create a category for **SharePoint** -- since that seemed to suggest creating categories for other Microsoft products as well (e.g. Visual Studio) and thus would eventually lead to large number of categories that closely paralleled the tags on my content. I eventually decided that SharePoint-related posts constitute a significant ratio of my content and therefore decided to keep it. This may change in the future, but it's what I'm currently using.

My intent is that posts are typically associated with one or two categories (e.g. **SharePoint** + **Development**) and tags are used to further refine categories (e.g. **MOSS 2007** vs. **SharePoint 2010**). The important thing to realize about tags and categories in Subtext is that categories are implemented as a highly structured taxonomy (meaning you have to explicitly define the list of available categories) whereas tags are much more of a "folksonomy" (meaning you can dynamically "create" new tags when creating or updating a post).

Here is the helper method that adds the tags to each post:

```
        private static void AppendBlogPostTags(
            HtmlDocument document,
            StringBuilder buffer)
        {
            string[] tags = GetPostTags(document);

            if (tags == null || tags.Length < 1)
            {
                return;
            }

            buffer.Append("<div class='tags'>");
            buffer.AppendFormat("<h3>Tags</h3>");
            buffer.Append("<ul>");

            foreach (string tag in tags)
            {
                buffer.Append("<li>");

                string newTag = GetNewTag(tag);

                buffer.AppendFormat(
                    "<a rel='tag' href='/blog/jjameson/tags/{0}'>{0}</a>",
                    newTag);

                buffer.Append("</li>");
            }

            buffer.Append("</ul>");
            buffer.Append("</div>");
        }
```

The **GetPostTags** method simply uses the now familiar parsing functionality of **HtmlDocument** to extract the list of tags from the offline post file:

```
        private static string[] GetPostTags(
            HtmlDocument document)
        {
            HtmlNode postTagsSection = document.DocumentNode.SelectSingleNode(
                "//div[@class='post-tags']");

            if (postTagsSection == null)
            {
                return null;
            }

            List<string> tags = new List<string>();

            IEnumerable<HtmlNode> postTags = postTagsSection.Descendants("a");

            foreach (HtmlNode link in postTags)
            {
                tags.Add(link.InnerText);
            }

            return tags.ToArray();
        }
```

I created the **GetNewTag** method (called from the **AppendBlogPostTags** method above) when I realized I should have used a different tag on my MSDN blog (i.e. **SharePoint 2010** -- instead of **SharePoint Server 2010**):

```
        private static string GetNewTag(
            string oldTag)
        {
            switch (oldTag)
            {
                case "SharePoint Server 2010":
                    return "SharePoint 2010";

                default:
                    return oldTag;
            }
        }
```

In order to populate the list of categories for each post, I created a simple mapping, illustrated below:

{{< table class="small" caption="Tag-to-Category Mapping" >}}

| Tag | Category |
| --- | --- |
| Core Development | Development |
| Debugging | Development |
| Infrastructure | Infrastructure |
| ISA Server | Infrastructure |
| MOSS 2007 | SharePoint |
| My System | My System |
| Personal | Personal |
| PowerShell | N/A |
| SharePoint Server 2010 | SharePoint |
| Silverlight | Development |
| Simplify | My System |
| SQL Server | N/A |
| TFS | Development |
| Toolbox | My System |
| Tugboat | SharePoint |
| Virtualization | Infrastructure |
| Visual Studio | Development |
| Web Development | Development |
| WCF | Development |
| Windows 7 | Infrastructure |
| Windows Server | Infrastructure |
| Windows Vista | Infrastructure |
| WSS v2 | SharePoint |
| WSS v3 | SharePoint |
| WSUS | Infrastructure |

{{< /table >}}

> **Note**
>
> Since posts tagged with **PowerShell** or **SQL Server** could fall into different categories depending on their specific content (e.g. **Infrastructure** or **Development**), I decided not to use these tags for mapping (and instead rely on categories being derived from other tags on the same post).

This mapping is implemented in the **MapTagToCategory** method, which is called from the **FillPostCategories** method:

```
        private static void FillPostCategories(
            BlogMLBlog blog,
            BlogMLPost post,
            HtmlDocument document)
        {
            string[] tags = GetPostTags(document);

            if (tags == null || tags.Length < 1)
            {
                return;
            }

            foreach (string tag in tags)
            {
                string categoryTitle = MapTagToCategory(tag);

                if (string.IsNullOrEmpty(categoryTitle) == false)
                {
                    BlogMLCategoryReference category = EnsureCategory(
                        blog,
                        categoryTitle,
                        post.DateCreated);

                    AddCategoryIfNotAlreadyReferenced(
                        post.Categories,
                        category);
                }
            }
        }

        private static string MapTagToCategory(
            string tag)
        {
            switch (tag)
            {
                case "Core Development":
                case "Debugging":
                case "Silverlight":
                case "TFS":
                case "Visual Studio":
                case "Web Development":
                case "WCF":
                    return "Development";

                case "Infrastructure":
                case "ISA Server":
                case "Virtualization":
                case "Windows 7":
                case "Windows Server":
                case "Windows Vista":
                case "WSUS":
                    return "Infrastructure";

                case "My System":
                case "Simplify":
                case "Toolbox":
                    return "My System";

                case "Personal":
                    return "Personal";

                case "MOSS 2007":
                case "SharePoint Server 2010":
                case "Tugboat":
                case "WSS v2":
                case "WSS v3":
                    return "SharePoint";

                case "PowerShell":
                case "SQL Server":
                    return null;

                default:
                    throw new ArgumentException("Unexpected tag");
            }
        }
```

> **Tip**
>
> Depending on the complexity of your taxonomy, you may need a more robust mapping implementation than the one I've shown here. For example, on the Agilent project, I created an Excel workbook with a separate worksheet for each property that needed to be mapped during the migration. These worksheets were subsequently "scrubbed" by one of the Agilent team members and then loaded into SharePoint lists that served as "dynamic lookup tables" during the content migration process.

### Step 5: Export the comments for each post

Migrating the post comments was a bit trickier than I originally anticipated. This is because -- contrary to what I expected -- the comments are not included in the offline file for each post.

I discovered the MSDN blogs (and perhaps all blogs based on the Telligent platform) load the comments asynchronously (i.e. using Ajax). In other words, when you request an individual blog post, some JavaScript runs that requests the comments for the post and subsequently injects these into the HTML.

Fortunately, after a couple of hours of tinkering I figured out how to "Web scrape" the Ajax request that loads the comments for a particular post. Unfortunately it is not as simple as issuing an HTTP GET or POST with a couple of parameters. If you don't get the HTTP headers right, then Telligent simply "barfs on you" (meaning it returns an error).

In order to spoof the Ajax requests to download blog comments, I implemented what essentially amounts to a "replay hack." Like the previous steps, let's start with a quick look at the updates to `Main`...

```
        static void Main(string[] args)
        {
            Uri oldBlogBaseUrl = new Uri("http://blogs.msdn.com/b/jjameson");

            ...

            const string commentsFolder =
                @"C:\NotBackedUp\Temp\MSDN-blog\Comments";

            ...

            BlogMLBlog blog = new BlogMLBlog();

            ...

            ExportComments(blog, commentsFolder, oldBlogBaseUrl);
        }
```

Similar to **ExportPosts**, the **ExportComments** method iterates the collection of posts, calls another method to export the post comments to an offline file, and then processes the offline file to load the comments into the BlogML document:

```
        private static void ExportComments(
            BlogMLBlog blog,
            string commentsFolder,
            Uri oldBlogBaseUrl)
        {
            foreach (BlogMLPost post in blog.Posts)
            {
                ...

                Uri originalPostUrl = GetOriginalPostUrl(
                    post,
                    oldBlogBaseUrl);

                string relativePath = originalPostUrl.AbsolutePath.Substring(
                    oldBlogBaseUrl.AbsolutePath.Length + 1);

                relativePath = relativePath.Replace('/', '\\');

                string commentsFilename = Path.Combine(
                    commentsFolder,
                    relativePath);

                Debug.Assert(commentsFilename.EndsWith(".aspx") == true);

                commentsFilename = Path.ChangeExtension(
                    commentsFilename,
                    ".html");

                ExpostPostComments(originalPostUrl, commentsFilename);

                if (File.Exists(commentsFilename) == true)
                {
                    FillPostComments(
                        blog,
                        post,
                        commentsFilename);
                }
            }
        }
```

**ExportPostComments** checks to see if an offline comments file exists for the specified post (meaning the comments have already been downloaded) and, if not, attempts to download any comments:

```
        private static void ExportPostComments(
            Uri originalPostUrl,
            string commentsFilename)
        {
            if (File.Exists(commentsFilename) == false)
            {
                Console.WriteLine(
                    "Downloading comments for post {0}...",
                    originalPostUrl);

                string archiveFolder = Path.GetDirectoryName(commentsFilename);

                if (Directory.Exists(archiveFolder) == false)
                {
                    Directory.CreateDirectory(archiveFolder);
                }

                string feedbackHtml = GetFeedbackHtmlForPost(originalPostUrl);

                if (string.IsNullOrEmpty(feedbackHtml) == true)
                {
                    return;
                }

                using (StreamWriter writer = File.CreateText(commentsFilename))
                {
                    writer.Write(feedbackHtml);
                }
            }
        }
```

The **GetFeedbackHtmlForPost** method is where the "magic" happens for Web scraping the fake Ajax request used to retrieve the comments:

```
        private static string GetFeedbackHtmlForPost(
            Uri postUrl)
        {
            WebClient client = new WebClient();

            client.Headers.Add("Accept: */*");
            client.Headers.Add("Content-Type: application/x-www-form-urlencoded; charset=utf-8");
            client.Headers.Add("Referer: " + postUrl);
            client.Headers.Add("Accept-Language: en-us");
            client.Headers.Add("Accept-Encoding: gzip, deflate");
            client.Headers.Add("User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0)");
            //client.Headers.Add("Host: blogs.msdn.com");
            //client.Headers.Add("Content-Length: 1103");
            //client.Headers.Add("Connection: Keep-Alive");
            client.Headers.Add("Pragma: no-cache");
            client.Headers.Add(
"Cookie: s_cc=true; s_sq=%5B%5BB%5D%5D; ANON=A=E7B0717DB3038FFD349B1318FFFFFFFF&E=bb6&W=1; NAP=V=1.9&E=b5c&C=0HrFGCvEgFAZZKizfEhC2PUQmuIwQ9q9CPrLHHZp10525aFhJTTAOA&W=1; omniID=1316546319777_a3e1_22fc_9952_251725e6a494; AuthorizationCookie=d114cf6b-a8d3-4af4-869b-742773394143; mstcid=4d54d9b");

            string delayedFeedbackPostData =
"ctl00$content$ctl00$ctl00=custom%3Aid%3Dfragment-52319%26renderFromCurrent%3DTrue%26callback_control_id%3Dctl00%2524content%2524ctl00%2524w_52319%2524_d49195%2524ctl00%2524ctl00%2524ctl02%2524ctl05%2524DelayedFeedbackList%26callback_argument%3Dundefined&ctl00_content_ctl00_w_52320__d49195_ctl00_ctl00_ctl00_ctl05_bpCommentForm_ctl05_invisibleCaptcha_answer=4&ctl00_content_ctl00_w_52320__d49195_ctl00_ctl00_ctl00_ctl05_bpCommentForm_ctl05_invisibleCaptcha_encrypted=ZkbskFnzZtMiphWuRsfNv3GiqbzZ%2FMNBpDkbrxKoXf0%3D&__VIEWSTATE=%2FwEPDwUKMTM5OTAxNDYyNWRkvI787cXLvh9XHlDReEjVejxzcZ4%3D&MSTWMenu=es&ctl00$content$ctl00$w_52312$_d49195$ctl00$ctl01$ctl01$DatingBackTo=-1&SearchTypeRadio=ThisBlog&ctl00$content$ctl00$w_52317$_d49195$ctl00$ctl15$ctl02=value%3A%253Ca%2520href%253D%2522http%253A%252F%252Fblogs.msdn.com%252Fb%252Fjjameson%252Farchive%252Ftags%252FMy%252BSystem%252F%2522%2520rel%253D%2522tag%2522%253EMy%2520System%253C%252Fa%253E%252C%2520%253Ca%2520href%253D%2522http%253A%252F%252Fblogs.msdn.com%252Fb%252Fjjameson%252Farchive%252Ftags%252FInfrastructure%252F%2522%2520rel%253D%2522tag%2522%253EInfrastructure%253C%252Fa%253E";

            try
            {
                byte[] data = Encoding.ASCII.GetBytes(delayedFeedbackPostData);

                byte[] compressedResponse = client.UploadData(postUrl, data);

                MemoryStream decompressedResponse = new MemoryStream();

                using (Stream responseStream = new MemoryStream(compressedResponse))
                {
                    GZipStream decompressedStream = new GZipStream(
                        responseStream,
                        CompressionMode.Decompress);

                    byte[] buffer = new byte[4096];
                    int numRead;
                    while ((numRead = decompressedStream.Read(buffer, 0, buffer.Length)) != 0)
                    {
                        decompressedResponse.Write(buffer, 0, numRead);
                    }
                }

                decompressedResponse.Position = 0;

                StreamReader reader = new StreamReader(decompressedResponse);
                string decompressedData = reader.ReadToEnd();
                Debug.Assert(decompressedData.StartsWith(
                    "s{'response':") == true);

                int index1 = decompressedData.IndexOf("<div");
                if (index1 == -1)
                {
                    return null;
                }

                int index2 = decompressedData.LastIndexOf(@"<\/div>");
                Debug.Assert(index2 > 0);

                index2 += @"<\/div>".Length;

                string feedbackHtml = decompressedData.Substring(
                    index1,
                    index2 - index1);

                feedbackHtml = feedbackHtml.Replace(@"\/", "/");
                feedbackHtml = feedbackHtml.Replace(@"\'", "'");
                feedbackHtml = feedbackHtml.Replace("&#39;", "'");

                feedbackHtml = feedbackHtml.Replace(
                    @"\r\n",
                    Environment.NewLine);

                feedbackHtml = feedbackHtml.Replace(@"\n", Environment.NewLine);

                feedbackHtml = feedbackHtml.Replace(
                    "<div style=\"clear:both;\"></div>",
                    string.Empty);

                feedbackHtml = feedbackHtml.Trim();

                return feedbackHtml;
            }
            catch (WebException ex)
            {
                Console.WriteLine(ex.Message);
                throw;
            }
        }
```

To create this method, I used Fiddler to inspect the Ajax request triggered when viewing an individual blog post. I then copied the HTTP headers from that sample request and pasted them into the code. Once I had a valid response being returned from Telligent, I wrote the code to decompress it (since the response is GZIP'ed) and parse the comments.

> **Tip**
>
> You can easily inspect/copy HTTP requests and responses using Fiddler, the **Network** tab in the Internet Explorer 9 developer tools, or with Firefox and the Firebug add-on.

> **Important**
>
> The cookie shown in the code above is only valid for a limited time. Consequently I needed to periodically get a new cookie and paste it into the code during the time I was developing this migration utility.

### Step 6: Save the BlogML document to an XML file

After running the migration utility at this point, the BlogML document contains all of the blog posts (including the categories, tags, and comments for each post) and is ready to be written to a file.

```
        static void Main(string[] args)
        {

            ...

            string blogExportFile = @"C:\NotBackedUp\Temp\"
                + "Random Musings of Jeremy Jameson-Export.xml";

            ...

            SaveBlogML(blog, blogExportFile);
        }
```

The **SaveBlogML** method is very simple...

```
        private static void SaveBlogML(
            BlogMLBlog blog,
            string filename)
        {
            BlogMLWriter writer = new BlogMLWriter(blog);

            using (XmlTextWriter xmlWriter = new XmlTextWriter(
                filename,
                Encoding.UTF8))
            {
                writer.Write(xmlWriter);
            }
        }
```

...however -- much to my surprise -- BlogML .NET doesn't actually include all of the functionality to write an instance of **BlogMLBlog** to an XML file. Instead, it provides you a base class (**BlogMLWriterBase**) that you need to derive from (and implement the abstract **InternalWriteBlog** method).

Fortunately, this wasn't an issue since I was able to copy the **BlogMLWriter** class from the Subtext source code and make a couple of tweaks to make it work for my migration tool.

### Step 7: Import the BlogML file

With the 5 MB XML file generated by my **ExportMsdnBlog** utility, I was almost ready to import the BlogML file into Subtext for the final time. However, I discovered a couple of issues during my testing:

1. When importing a blog post comment containing multiple paragraphs, extra line breaks appeared in the Subtext comment. I ended up modifying Subtext to avoid this issue. [I'll describe this change in a different post.]
2. When exporting blog posts from my MSDN blog, recall that I used the monthly summary pages to get the list of posts. These summary pages do not list the posts in the order they were published, but rather in reverse order (in other words, showing the more recent posts first). When importing the posts, my preference was to add them in the order in which they would have been if I had actually created them originally in Subtext (a minor nitpick for sure, but still one I preferred to avoid if it didn't require significant effort).

To resolve this second issue, I simply needed to sort the `<post>` elements in the BlogML file by the `date-created` attribute. To do this, I created an XSLT file in Visual Studio and added a custom template for the `<posts>` elements (i.e. the container for the `<post>` elements). I also needed to specify the XML namespace in order to get this to work as expected (since the BlogML file specifies a default namespace):

```
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:b="http://www.blogml.com/2006/09/BlogML"
    xmlns="http://www.blogml.com/2006/09/BlogML"
    exclude-result-prefixes="b">
    <xsl:output method="xml" indent="yes"/>

  <xsl:template match="b:posts">
    <posts>
      <xsl:apply-templates select="b:post">
        <xsl:sort select="@date-created" />
      </xsl:apply-templates>
    </posts>
  </xsl:template>

  <xsl:template match="@* | node()">
    <xsl:copy>
      <xsl:apply-templates select="@* | node()"/>
    </xsl:copy>
  </xsl:template>
</xsl:stylesheet>
```

In hindsight, I suppose another alternative would have been to customize the **WritePosts** method of the **BlogMLWriter** class to use LINQ to select the objects from the **PostCollection** in a specific order (instead of the simple `foreach` loop currently specified in that method). If this was something I expected to use on an ongoing basis, that would have been a preferable approach. However, sorting the file using XSLT was good enough for a "one-off" scenario.

At this point I rebuilt my Subtext database (to get a fresh instance with no data whatsoever), recreated my blog using the Subtext admin functionality (which fortunately goes very quickly), and imported the BlogML file. A few minutes later, I verified that all 300+ blog posts were successfully migrated.

### What about images and attachments?

The astute reader may be wondering about the images and attachments included in my original MSDN blog posts. In other words, I haven't mentioned anything about how my utility migrates these files from Telligent. That is because it doesn't.

Instead I chose to migrate my blog images and attachments manually, for a couple of reasons:

1. The number of attachments included in my blog posts was rather low (and thus I estimated it would take substantially less time to migrate these manually than trying to automate it through code).
2. The way images are stored in Subtext is fundamentally very different from the way they are stored in Telligent. In particular, the way the resized images are created and stored is something I didn't even want to attempt to automate through my own code. If you've followed my blog for a while, then you have probably noticed that I prefer to show images at reduced sizes within the context of a post and provide links to view the corresponding full-sized images (a trick I picked up years ago from the Microsoft Technet site).

To help expedite this manual migration process, I did end up using the Html Agility Pack to quickly generate the list of posts containing images. That way I didn't have to review each and every one of the 300+ migrated posts when looking for images to migrate.

This is the final point I want to make regarding these kinds of content migration scenarios. While it would be nice to achieve 100% automated migration from one system to another, it is almost certainly not worth the effort (and time) required to do this through code. Of course, this depends largely on how much content you are migrating and how long the "manual" portion of the migration will take. Always keep in mind the concept of "business value" when writing migration code like this.

