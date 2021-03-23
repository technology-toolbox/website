---
title: Errors running the Subtext blog engine -- and what I did about them (a.k.a. Building TechnologyToolbox.com, part 15)
date: 2012-01-23T13:29:43-07:00
excerpt:
  I've been running my new blog on Subtext for almost 4 months and encountered
  some issues with Subtext that required a few tweaks here and there. In this
  post, I describe the errors I've seen and my recommendations for addressing
  them.
aliases:
  [
    "/blog/jjameson/archive/2012/01/23/building-technologytoolbox-com-part-15.aspx",
  ]
draft: true
categories: ["Development", "My System"]
tags: ["Subtext"]
---

I've been running my new blog on Subtext for almost 4 months and overall I'm
very satisfied with it. However, as I mentioned in a
[previous post](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-1),
I encountered some issues with Subtext that required a few tweaks here and
there.

Some of the changes I made to Subtext were necessary to get the blog pages to
render as I wanted them to ("enhancements" if you will), while other changes
were required to mitigate errors on the site (i.e. "bug fixes"). In this post,
I'll describe the latter, since these will be of greater interest to a broader
audience.

As I mentioned in
[yesterday's post](/blog/jjameson/2012/01/22/building-technologytoolbox-com-part-14),
ELMAH is used to log errors on the Technology Toolbox site and I've configured
it to send an email whenever an unhandled exception occurs on the site.
Consequently I've received somewhere in the neighborhood of 500 messages in the
past few months as a result of errors on the site.
[Thankfully these email messages are organized automatically into various folders, courtesy of [rules I have configured in Outlook](/blog/jjameson/2010/01/04/managing-email-effectively).]

A few of these errors are my fault (in other words, they occurred either in code
that I developed or due to configuration issues) but the vast majority result
from issues in the Subtext blog engine. It's also worth emphasizing that many of
the errors are duplicates of the same issue (e.g. the IdenticonHandler.ashx
error that I describe below) and many of them are caused by people trying to
hack the site.

I should also mention that _some_ - but not _all_ -- of the errors reported by
ELMAH for the blog pages appear in the Subtext error log as well (i.e. Subtext
admin dashboard → **Stats** → **Error Log**). Honestly, I've never bothered to
investigate why some of the errors reported by ELMAH are not detected by
Subtext.

Consequently, I tend to rely much more on the ELMAH logs -- and only
occasionally check the error log built into Subtext. Also, most of the errors
reported in the Subtext error log for TechnologyToolbox.com are due to malicious
people trying to hack the site (e.g. "Could not decrypt the authentication
cookie.") which I generally consider to just be "noise."

{{< div-block "note" >}}

> **Note**
>
> Sometimes it is important to investigate the attempts to hack your site, but
> -- assuming you actually want to accomplish something useful with your time --
> you typically have to chock these up to a fact of life when hosting a site on
> the Internet. Make sure the code you write is secure, and don't be suprised
> when you see evidence that hackers are trying to find ways to torment you.

{{< /div-block >}}

The errors below are not listed in the order in which they occurred (or fixed).
Rather, the list is loosely ordered by the frequency of occurence and severity
of the underlying issue (typically evaluated in terms of impact on user
experience). In other words, I have triaged the issues so that if you are
running Subtext on your own site, you would most likely focus on the items that
appear first.

### Lucene.Net.QueryParsers.ParseException: Encountered "..." at line 1, column ... Was expecting one of: &lt;EOF&gt; &lt;AND&gt; ...

Having read a little about Subtext before implementing it, I knew that it uses
[Lucene.Net](http://en.wikipedia.org/wiki/Lucene.net) to support searching
across blog posts. What I didn't know -- at least not until I encountered this
error -- is that your blog may "blow chunks" as a result of Lucene.Net even if
your blog skin does not show the Subtext search box (for example, Technology
Toolbox currently uses Google Site Search).

The problem is due to the following code in **SubtextMasterPage**:

```C#
        public void InitializeControls(ISkinControlLoader controlLoader)
        {
            ...
                    var query = Query;
                    if(!String.IsNullOrEmpty(query))
                    {
                        var searchResults = SearchEngineService.Search(query, 5, Blog.Id, entryId);
                        if(searchResults.Any())
                        {
                            AddMoreResultsControl(searchResults, controlLoader, apnlCommentsWrapper);
                        }
                    }
            ...
        }
```

Since I really don't want the Subtext master page to "magically" suggest other
blog posts to users (based on what they searched for using, say, Google), I
updated the code as follows:

```C#
        public void InitializeControls(ISkinControlLoader controlLoader)
        {
            ...

                    // Bug 1053: Allow errors in
                    // Lucene.Net.QueryParsers.QueryParser to be avoided by
                    // disabling the full text search engine in Web.config
                    if (FullTextSearchEngineSettings.Settings.IsEnabled == true)
                    {
                        var query = Query;
                        if(!String.IsNullOrEmpty(query))
                        {
                            var searchResults = SearchEngineService.Search(query, 5, Blog.Id, entryId);
                            if(searchResults.Any())
                            {
                                AddMoreResultsControl(searchResults, controlLoader, apnlCommentsWrapper);
                            }
                        }
                    }
            ...
        }
```

...and subsequently disabled the Subtext search engine via the Web.config file:

```XML
  <FullTextSearchEngineSettings
    type="Subtext.Framework.Configuration.FullTextSearchEngineSettings, Subtext.Framework">
    <IsEnabled>false</IsEnabled>
    ...  </FullTextSearchEngineSettings>
```

**Recommendation:** Implement the fix described above to prevent "automatic
search results" from being added by the Subtext master page -- or try upgrading
to a newer version of Lucene.Net to see if it still spews errors when the HTTP
referrer specifies something like:

```
http://webcache.googleusercontent.com/search?q=cache:oYV0jIrX76cJ:www.technologytoolbox.com/blog/jjameson/archive/2011/05.aspx+%22visual+studio%22+%22new+XsltListViewWebPart%22&cd=33&hl=ru&ct=clnk&gl=lv
```

### System.Security.Cryptography.CryptographicException: Padding is invalid and cannot be removed.

I encountered this error shortly after migrating my blog to Subtext and it
typically occurred dozens of times per day. The inner exception reported by
ELMAH turns out to be a little misleading. When you look at the next exception
up the stack, the underlying problem is revealed:

{{< log-excerpt >}}

Subtext.Web.Controls.Captcha.CaptchaExpiredException: Captcha image expired,
probably due to recompile making the key out of synch.

{{< /log-excerpt >}}

The CAPTCHA control included in Subtext wreaks havoc on your site if the
Googlebot discovers a CAPTCHA image, adds it to its index, and subsequently
attempts to crawl (or re-crawl) the image at some later point in time (after the
application pool has recycled).

The problem is that -- at least in Subtext 2.5.2.0 -- the key and initialization
vector (IV) used in the **CaptchaBase** class are randomly generated each time
the Web application starts up. What I found particularly interesting is at some
point there was intent to generate these values once per environment. At least
that is what I inferred from the following comment:

```C#
        static SymmetricAlgorithm InitializeEncryptionAlgorithm()
        {
            SymmetricAlgorithm rijaendel = Rijndael.Create();
            //TODO: We should set these values in the db the very first time this code is called and load them from the db every other time.
            rijaendel.GenerateKey();
            rijaendel.GenerateIV();
            return rijaendel;
        }
```

To fix this issue, I changed the code as follows:

```C#
        static SymmetricAlgorithm InitializeEncryptionAlgorithm()
        {
            SymmetricAlgorithm rijaendel = Rijndael.Create();

            // To avoid numerous CryptographicException ("Padding is invalid and cannot
            // be removed.") errors from occurring when Google crawls the site, use a
            // static key and initialization vector for the CAPTCHA controls. If these
            // settings are not specified, then a random key and IV are created each time
            // the Subtext site starts up (which can cause CryptographicException errors
            // to occur when Google requests images/services/CaptchaImage.ashx and
            // specifies a query string that is no longer valid, because the app pool
            // recycled between the time Google discovered the CaptchaImage.ashx
            // reference and the time it actually initiates the request for
            // CaptchaImage.ashx). If specified, the encryption key and IV are expected
            // to be Base64 encoded.
            string key = ConfigurationSettings.AppSettings["Captcha.Encryption.Key"];
            string iv = ConfigurationSettings.AppSettings["Captcha.Encryption.IV"];

            if (string.IsNullOrEmpty(key) == false)
            {
                if (string.IsNullOrEmpty(iv) == true)
                {
                    throw new ConfigurationErrorsException(
                        "Captcha.Encryption.IV application setting must be"
                        + " specified when Captcha.Encryption.Key setting is"
                        + " specified.");
                }

                rijaendel.Key = Convert.FromBase64String(key);
                rijaendel.IV = Convert.FromBase64String(iv);
            }
            else
            {
                if (string.IsNullOrEmpty(iv) == false)
                {
                    throw new ConfigurationErrorsException(
                        "Captcha.Encryption.Key application setting must be"
                        + " specified when Captcha.Encryption.IV setting is"
                        + " specified.");
                }

                //TODO: We should set these values in the db the very first time this code is called and load them from the db every other time.
                rijaendel.GenerateKey();
                rijaendel.GenerateIV();
            }

            return rijaendel;
        }
```

Then I generated a key and IV and specified the Base64-encoded values in my
Web.config file:

```XML
  <appSettings>
    <!--
      To avoid numerous CryptographicException ("Padding is invalid and cannot
      be removed.") errors from occurring when Google crawls the site, use a
      static key and initialization vector for the CAPTCHA controls. If these
      settings are not specified, then a random key and IV are created each time
      the Subtext site starts up (which can cause CryptographicException errors
      to occur when Google requests images/services/CaptchaImage.ashx and
      specifies a query string that is no longer valid, because the app pool
      recycled between the time Google discovered the CaptchaImage.ashx
      reference and the time it actually initiates the request for
      CaptchaImage.ashx). If specified, the encryption key and IV are expected
      to be Base64 encoded.
    -->
    <add key="Captcha.Encryption.Key" value="ppujW5AxO9oz...="/>
    <add key="Captcha.Encryption.IV" value="eAP5g1WGujEF...=="/>
    ...
  </appSettings>
```

This is admittedly somewhat of a hack (I didn't develop any "tool" to create
some values for the Web.config file, much less provide a way to save these in
configuration without editing the file directly). However, it was a simple
workaround to a rather annoying error -- allowing me to redirect my efforts
elsewhere.

{{< div-block "note important" >}}

> **Important**
>
> Since the Googlebot will likely have added "bad" CAPTCHA images to its index
> (i.e. images based on a random key and IV), it may take several months before
> you stop seeing this error. The good news is that Google will eventually "give
> up" and remove the old CAPTCHA URLs from the index. The new CAPTCHA image URLs
> (based on the fixed key and IV specified in Web.config) will no longer
> generate the errors.
>
> In my case, these errors stopped occurring around the middle of December (and
> I switched to a fixed key and IV in early October).

{{< /div-block >}}

**Recommendation:** Implement the fix described above and then be patient.

### System.Security.SecurityException: Request failed. (IdenticonHandler.ashx)

If, like me, you run Subtext in a **Medium** trust configuration then you've
likely experienced these errors. Whether or not you were aware of these errors
likely depends on whether or not your blog skin displays an identicon next to
post comments. If the skin doesn't display identicons (e.g. the Technology
Toolbox skin does not) then the only time you'll see this error is when a blog
administrator browses to the Subtext admin dashboard.

This was actually one of the last errors I fixed on the Technology Toolbox site.
Since it did not have any impact on users, there was little motivation to fix
it. However, when I eventually grew tired of the errors being reported, I set
off on implementing a fix.

That's when I discovered that [Phil](http://haacked.com/) had already
investigated this bug (reported by somebody else) and implemented a fix (which
completely rips out the IdenticonHandler). Consequently I merged Phil's
changeset into my private branch of Subtext. Unfortunately, I wasn't able to
simply copy/paste his updates since I had previously tweaked portions of this
code to provide more customization options for displaying Gravatar images next
to post comments.

For the sake on not having to rehash an old topic, I'll just copy/paste my
check-in comments below:

{{< div-block "fst-italic" >}}

> Resolve errors with IdenticonHandler.ashx when running in "Medium" trust (by
> removing Identicon code in Subtext and instead relying on Gravatar service to
> render these images). This is very similar to Subtext revision r4237
> ([http://code.google.com/p/subtext/source/detail?r=4237](http://code.google.com/p/subtext/source/detail?r=4237))
> but varies somewhat to account for differences in the expected behavior (for
> example, do not show Gravatar image when no email address or default image is
> specified).

{{< /div-block >}}

Here is what my version of the **GravatarService** class looks like:

```C#
    public class GravatarService
    {
        public GravatarService(NameValueCollection settings)
            : this(settings["GravatarUrlFormatString"],
                settings.GetBoolean("GravatarEnabled"))
        {
        }

        public GravatarService(string urlFormatString, bool enabled)
        {
            UrlFormatString = urlFormatString;
            Enabled = enabled;
        }

        public bool Enabled { get; private set; }

        public string UrlFormatString { get; private set; }

        private static string EncodeDefaultImage(
            string defaultImage)
        {
            if (string.IsNullOrEmpty(defaultImage) == true)
            {
                return defaultImage;
            }

            if (defaultImage == "404"
                || defaultImage == "identicon"
                || defaultImage == "mm"
                || defaultImage == "monsterid"
                || defaultImage == "retro"
                || defaultImage == "wavatar")
            {
                return defaultImage;
            }

            defaultImage = HttpHelper.ExpandTildePath(defaultImage);

            Uri defaultImageUrl = new Uri(
                defaultImage,
                UriKind.RelativeOrAbsolute);

            if (defaultImageUrl.IsAbsoluteUri == false)
            {
                if (HttpContext.Current == null)
                {
                    string message = string.Format(
                        CultureInfo.CurrentCulture,
                        "Unable to convert default image relative URL ({0}) to"
                            + " corresponding absolute URL because "
                            + " HttpContext.Current is null.",
                            defaultImage);

                    throw new InvalidOperationException(message);
                }

                defaultImageUrl = new Uri(
                    HttpContext.Current.Request.Url,
                    defaultImageUrl);
            }

            defaultImage = HttpUtility.UrlEncode(
                defaultImageUrl.AbsoluteUri);

            return defaultImage;
        }

        public string GenerateUrl(string email)
        {
            return GenerateUrl(email, (string) null);
        }

        public string GenerateUrl(string email, Uri defaultImage)
        {
            return GenerateUrl(
                email,
                defaultImage != null ? defaultImage.ToString() : string.Empty);
        }

        public string GenerateUrl(string email, string defaultImage)
        {
            // Even if no email address is specified, Gravatar can still be used
            // to return an image (e.g. if default image is "mm" or the URL of an
            // image)
            string emailForUrl = email ?? string.Empty;
            emailForUrl = emailForUrl.ToLowerInvariant();

            emailForUrl =
(FormsAuthentication.HashPasswordForStoringInConfigFile(
    emailForUrl, "md5") ?? string.Empty).ToLowerInvariant();

            defaultImage = EncodeDefaultImage(defaultImage);

            return String.Format(CultureInfo.InvariantCulture, UrlFormatString,
                emailForUrl, defaultImage);
        }
    }
```

In the **Comments** control, I modified the logic for displaying Gravatar images
next to post comments:

```C#
protected void CommentsCreated(object sender, RepeaterItemEventArgs e)
{
    if(...)
    {
        ...
        if(feedbackItem != null)
        {
            ...
            if(_gravatarService.Enabled)
            {
                var gravatarImage = e.Item.FindControl("GravatarImg") as Image;
                if(gravatarImage != null)
                {
                    //This allows a host-wide setting of the default gravatar image.
                    string defaultImage = ConfigurationManager.AppSettings["GravatarDefaultImage"];

                    //This allows per-skin configuration of the default gravatar image.
                    foreach (string attributeKey in gravatarImage.Attributes.Keys)
                    {
                        if (string.Compare(
                                attributeKey,
                                "PlaceHolderImage",
                                StringComparison.OrdinalIgnoreCase) == 0)
                        {
                            defaultImage = gravatarImage.Attributes["PlaceHolderImage"];
                            gravatarImage.Attributes.Remove("PlaceHolderImage");
                            break;
                        }
                    }

                    gravatarImage.ImageUrl = _gravatarService.GenerateUrl(
                        feedbackItem.Email,
                        defaultImage);

                    // Only change the Visible property of the image to
                    // true when an email address is specified in the
                    // comment. This allows a skin to specify a default
                    // Gravatar image (e.g.
                    // PlaceHolderImage="~/Skins/TechnologyToolbox1/Images/Silhouette-1.jpg")
                    // but also choose to hide the default image when
                    // no email address is available to retrieve a
                    // Gravatar (i.e. by specifying Visible="false" on
                    // the Image control).
                    if (string.IsNullOrEmpty(feedbackItem.Email) == false)
                    {
                        gravatarImage.Visible = true;
                    }
                }
            }
            ...
        }
    }
}
```

As noted in my comments in the code above, this allows greater flexibility in
Subtext blog skins. For example, this is what is currently specified in the
custom skin for the Technology Toolbox site (in Comments.ascx):

```ASP.NET
<div id="postComments">
  <h3>Comments</h3>
  ...
  <asp:Repeater ...>
    ...
    <ItemTemplate>
      <li ...>
        ...
        <div class="avatar">
          <caelum:BorderlessImage runat="server" ID="GravatarImg"
            AlternateText="Gravatar" Visible="False"
            PlaceHolderImage="~/Skins/TechnologyToolbox1/Images/Silhouette-1.jpg"
            Height="72" Width="72" /></div>
         ...
       </li>
    </ItemTemplate>
    ...
  </asp:Repeater>
</div>
```

The following post shows an example of comments that specify email addresses
(both with and without a corresponding Gravatar):

> [https://www.technologytoolbox.com/blog/jjameson/archive/2011/10/27/building-technologytoolbox-com-part-1.aspx](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-1)

The following post shows an example where users added comments but did not
specify their email addresses:

> [https://www.technologytoolbox.com/blog/jjameson/archive/2011/02/25/claims-login-web-part-for-sharepoint-server-2010.aspx](/blog/jjameson/2011/02/25/claims-login-web-part-for-sharepoint-server-2010)

Without the code changes in Subtext that I show above, your site can't address
both scenarios. Then again, maybe you aren't quite as "nitpicky" as I am ;-)

**Recommendation:** Merge Subtext revision
[r4237](http://code.google.com/p/subtext/source/detail?r=4237) into your version
of Subtext (optionally merging my updates to the **GravatarService** and
**Comments** classes to provide more customization options for Gravatar images).

### System.Security.SecurityException: That assembly does not allow partially trusted callers.

I mentioned in the previous section that I am currently running Subtext in a
**Medium** trust configuration. Consequently when I attempted to upload a zip
file to a gallery, Subtext threw up all over my nice, clean shoes. [I hope those
of you out there running in **Full** trust realize just how easy you've got it.
That is, of course, until somebody exploits a security hole in your solution or
in ASP.NET itself.]

After spending a few minutes investigating the bug, I found the culprit to be
ICSharpCode.SharpZipLib.dll. A little more digging and I discovered the latest
version of this assembly has the **AllowPartiallyTrusterCallers** assembly
attribute.

Woohoo! I love bugs that take less than an hour to resolve.

**Recommendation:** Download the latest version of ICSharpCode.SharpZipLib.dll,
overwrite the previous version in the Subtext solution, and rebuild. Then go
celebrate your bug-bashing victory by pouring yourself a glass of Maker's Mark.

### System.Security.SecurityException: Request for the permission of type 'System.Configuration.ConfigurationPermission, ...' failed.

Here's another error that only occurs when running Subtext in a **Medium** trust
configuration.

As I mentioned in a
[previous post](/blog/jjameson/2011/10/18/introducing-technologytoolbox-com), I
was initially hesitant about having both "tags" and "categories" for blog posts
(since [my old MSDN blog](http://blogs.msdn.com/b/jjameson) only used tags).
Therefore I started out only using categories (in other words, when migrating
posts from my MSDN blog, each tag in Telligent was converted to a category in
Subtext). Shortly thereafter, I decided it would be better to use _both_
categories and tags in Subtext. Consequently I ended up deleting the categories
I had previously created and starting over with a much smaller number. [Tags
from Telligent were essentially migrated one-for-one to tags in Subtext. The
original tags were also used to determine a higher level "grouping" which became
the categories in Subtext.]

However, the original categories had already been discovered by Google.
Consequently, when someone -- or the Googlebot itself -- attempted to view one
of the deleted categories (e.g.
https://www.technologytoolbox.com/blog/jjameson/category/19.aspx) an error would
occur.

While you would expect an HTTP 404 error (since the specified category does not
exist), in a **Medium** trust configuration, Subtext 2.5.2.0 actually throws a
**SecurityException**:

> System.Security.SecurityException: Request for the permission of type
> 'System.Configuration.ConfigurationPermission, System.Configuration,
> Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' failed.

The problem is in the **OnLoad** method of the **CategoryEntryList** class. To
fix this bug, I changed it to the following:

```C#
        protected override void OnLoad(EventArgs e)
        {
            ...
            if(...)
            {
                ...
                LinkCategory lc = Cacher.SingleCategory(SubtextContext);
                ...
                if(lc == null)
                {
                    // Bug 1054:
                    // When running under Medium trust, calling
                    // HttpHelper.SetFileNotFoundResponse() causes an
                    // exception while attempting to read the
                    // system.web/customErrors section of the Web.config
                    // file ("System.Security.SecurityException: Request for the
                    // permission of type
                    // 'System.Configuration.ConfigurationPermission,
                    // System.Configuration, Version=2.0.0.0, Culture=neutral,
                    // PublicKeyToken=b03f5f7f11d50a3a' failed.").
                    //
                    // Therefore, just throw 404 HttpException instead.
                    //
                    //HttpHelper.SetFileNotFoundResponse();
                    //return;

                    throw new HttpException(404, "Category not found.");
                }
                ...
            }
        }
```

{{< div-block "note important" >}}

> **Important**
>
> There are a few other places in the Subtext solution where the
> **HttpHelper.SetFileNotFoundResponse** method is called (and should probably
> be updated as well to avoid issues when running in **Medium** trust). However,
> at present, I haven't bothered to change those (since I am trying to minimize
> the number of changes that I make to my Subtext branch).

{{< /div-block >}}

### System.Web.HttpException: The file '/blog/jjameson/Services/Pingback.aspx' does not exist.

If the previous error corresponds to "low hanging fruit" then this one
definitely is somewhere near the top of the tree. Think extension ladders and
lots of setup just to pick the last few pieces of fruit from the trees way in
the back of the orchard.

The problem is, I don't think anyone has even looked at the "pingback"
functionality in Subtext in a long time. Perhaps not since it was .TEXT. [I'm
completely speculating since I didn't bother to dig into the source control
history for the Subtext solution.]

The point is that I did manage to repro the issue in my own environment (by
setting up another blog engine and creating a new post linking to one of my
posts on the Technology Toolbox site) but it certainly took more than an hour.

Again, for the sake on not having to rehash an old topic, I'll just copy/paste
my check-in comments below:

{{< div-block "fst-italic" >}}

> Fix numerous issues with pingback functionality in Subtext:\
>
> - Error in PROD (i.e. "System.Web.HttpException: The file
>   '/blog/jjameson/Services/Pingback.aspx' does not exist.")\
> - The "pingback" URL specified in the &lt;link&gt; head element should be an
>   absolute URL (not a relative URL) according to the Pingback 1.0 specification
>   (http://www.hixie.ch/specs/pingback/pingback)\
> - According to the current Subtext routing functionality, the "pingback" URL
>   needs to include the ID of the post (e.g.
>   "https://www.technologytoolbox.com/blog/jjameson/Services/Pingback/315.aspx")\
> - Method name was misspelled (i.e. "Notifiy" --&gt; "Notify")

{{< /div-block >}}

Then again, perhaps Pingback functionality and correct spelling (even in code)
just isn't all that important to you.

**Recommendation:** If you are genuinely interested in getting pingbacks working
in Subtext, then contact me. [Perhaps we can work out a trade for a small bottle
of Maker's Mark. Oh wait, there's probably some law against shipping a bottle of
bourbon across state lines. Well, I'm sure we can figure something out.]

### System.NullReferenceException: Object reference not set to an instance of an object.

Ah yes, the dreaded **NullReferenceException**. A surefire indicator of
insufficient "boundary checking" somewhere in the code.

{{< div-block "note" >}}

> **Note**
>
> In case you are new to .NET development, you should never experience a
> **NullReferenceException**. **ArgumentNullException**? Yes, those are "good"
> (sort of)...a **NullReferenceException** is definitely indicative of "bad"
> code.

{{< /div-block >}}

Honestly, this one hasn't occurred very often (only 16 times to date) -- and 10
of these occurred back-to-back over a period of 14 seconds back on December
19th. It looks like someone was "probing" for some security hole via an
automated script, because I see a bunch of requests for URLs like
"/blog/jjameson/aggbug/311.aspx".

The other instances occurred either in
`Subtext.Framework.Web.Handlers.SubtextPage.get_Blog()` or
`Subtext.Framework.Web.Handlers.SubtextPage.get_Repository()` -- and all but one
do _not_ specify an HTTP referrer. Again, this seems to point to hackers trying
to find a vulnerability in the site.

**Recommendation:** Keep an eye on this one and perhaps do some more research on
known security holes in Subtext (maybe older versions?).

### System.ArgumentOutOfRangeException: Index and length must refer to a location within the string. Parameter name: length

This error occurred twice back on November 4th and neither of them specified an
HTTP referrer. Then a slew of similar errors occurred on January 11th (but none
in between).

Hmmm...maybe I should do a quick WHOIS on 159.253.145.175 (the IP for the
January series)...

...interesting...it's neither China nor Russia. Actually WHOIS reports
"SoftLayer Dutch Holdings BV" and Country: United States. Aren't most hackers
supposedly outside the U.S.?

Well, maybe just somebody "innocently" screwing around with my site -- all under
the guise of some "holding" company.

**Recommendation:** Another one to keep an eye on.

### System.Data.SqlClient.SqlException: A network-related or instance-specific error occurred while establishing a connection to SQL Server.

Doh! It looks like the WinHost SQL Server environment was down when my site was
actively trying to service a request:

> System.Data.SqlClient.SqlException: A network-related or instance-specific
> error occurred while establishing a connection to SQL Server. The server was
> not found or was not accessible. Verify that the instance name is correct and
> that SQL Server is configured to allow remote connections. (provider: Named
> Pipes Provider, error: 40 - Could not open a connection to SQL Server)

Well, I'm not exactly paying for five 9's reliability. One of the reasons why I
picked WinHost is because they are _very_ cost effective. On the other hand, the
Technology Toolbox site is definitely a critical component for the company, so
I'd better watch for similar outages in the future.

Since this error has only happened one time (back on November 3rd), I'll call it
a "fluke" and let it slide.

**Recommendation:** Consider contacting the hosting provider if this occurs more
frequently in the future.

### System.Data.SqlClient.SqlException: Timeout expired...

Similar to the previous error:

> System.Data.SqlClient.SqlException: Timeout expired. The timeout period
> elapsed prior to completion of the operation or the server is not responding.

It happened once on November 8th and then twice on January 11th. From my notes,
I know that the November 8th instance was my own fault. I was doing a bulk
update on a number of blog posts (using a little utility that I wrote) and the
tool ended up creating a lock on the **subtext\_Content** table. Ugh.

See, this is exactly why Microsoft doesn't support directly querying SharePoint
databases. You have to be _very_ careful when accessing Production databases
(whether they are supporting SharePoint, Subtext, or some other system
entirely). Otherwise you could inadvertently cause problems (e.g. due to
locking).

I'm not sure what happened on January 11. All I know is that I wasn't running
any kind of bulk update at that time. I'll keep an eye out for future instances
of this error.

**Recommendation:** Before running any kind of utility in Production, ensure you
run it in the Test environment (that was recently
["refreshed" from Production](/blog/jjameson/2011/11/14/building-technologytoolbox-com-part-7))
and monitor locks in SQL Server while testing.

### System.Web.HttpException: This is an invalid script resource request.

One-time only...back on November 2nd.

No HTTP referrer.

**Recommendation:** Call it a rather lame attempt at hacking the site and move
on.

### System.Web.HttpException: This is an invalid webresource request.

While more frequent than the previous error, this error has only occurred 13
times (most recently on December 28th).

Looks like more pathetic hacking attempts.

**Recommendation:** Consider configuring a filter in ELMAH to weed out the
hacking "noise."

{{< div-block-start "note update" >}}

> **Update (2012-02-28)**
>
> Refer to the following post for more information on configuring ELMAH filters:
> {{< reference title="Filter ELMAH email messages to avoid getting spammed by hackers" linkHref="/blog/jjameson/2012/02/28/filter-elmah-email-messages-to-avoid-getting-spammed-by-hackers" linkText="https://www.technologytoolbox.com/blog/jjameson/archive/2012/02/28/filter-elmah-email-messages-to-avoid-getting-spammed-by-hackers.aspx" >}}

{{< div-block-end >}}

### System.Web.HttpException: A public action method 'RecordAggregatorView' could not be found...

Six of these in a span of approximately 3 seconds back on November 2nd:

> System.Web.HttpException: A public action method 'RecordAggregatorView' could
> not be found on controller 'Subtext.Web.Controllers.StatisticsController'.

Looks like another attempt at exploiting some "aggbug" vulnerability. Maybe it's
time to Google "aggbug" and see what's up...

Ah, now I get it...It looks like some remnant in Subtext from the old .TEXT days
(for tracking RSS views). This explains the absence of HTTP referrer. Perhaps
it's not hackers after all...still seems a little suspicious, but perhaps it
only occurs with certain RSS readers.

**Recommendation:** Ignore (for now).

### System.UriFormatException: Invalid URI: The URI scheme is not valid.

This error has only happened once (on December 19th).

A quick look at the HTTP\_REFERER field tells me all I need to know. Seriously,
whoever you are at 173.234.181.91, go away. I'm not interested in your
"enhancements" and I don't think people visiting my site would be either.

**Recommendation:** Ignore (hacker)

### System.InvalidOperationException: Sorry, but we cannot accept this comment.

Interesting...this error has only happened once (so far), but it points out one
of the methods hackers use when probing for security vulnerabilities. If you
lookup this error message in the Subtext source, you'll see it happens when
someone attempts to bypass the maximum length restrictions on the form fields
when adding a comment.

Bastard.

**Recommendation:** Ignore (hacker)

### System.Web.HttpException: The file '/blog/jjameson/user/CreateUser.aspx' does not exist.

Only two instances of this one -- first time on November 22nd and then a week
later on November 29th. Both from the same IP address (96.31.35.33).

Seriously, you're going to have to try harder than that. Perhaps you should try
some SQL-injection attacks instead?!

**Recommendation:** Ignore (hacker)

### System.Web.HttpException: The file '/blog/jjameson/post.aspx' does not exist.

Only two instances of this error (October 31st and January 3rd). In the first
instance, the HTTP referrer is actually my Feedburner URL. In the second one,
there is no HTTP referrer, but it appears to be from a crawler for a Russian
search engine I'm not familiar with (YandexBot).

**Recommendation:** Ignore (for now)

### System.ArgumentException: Invalid postback or callback argument. Event validation is enabled...

Only one instance of this (on November 1st), so it looks like it was probably
YAHA (yet another hacking attempt):

> System.ArgumentException: Invalid postback or callback argument. Event
> validation is enabled using &lt;pages enableEventValidation="true"/&gt; in
> configuration or &lt;%@ Page EnableEventValidation="true" %&gt; in a page. For
> security purposes, this feature verifies that arguments to postback or
> callback events originate from the server control that originally rendered
> them. If the data is valid and expected, use the
> ClientScriptManager.RegisterForEventValidation method in order to register the
> postback or callback data for validation.

**Recommendation:** Ignore it...or, better yet, take a moment to enjoy all of
the out-of-the-box "goodness" we get for free when it comes to security in
ASP.NET. [It really helps to offset what you feel whenever you see Microsoft
release a security patch for the .NET Framework.]
