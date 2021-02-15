---
title: "Custom CAPTCHA control for ASP.NET application and Subtext blog (a.k.a. Building TechnologyToolbox.com, part 16)"
date: 2012-01-25T05:44:15-07:00
excerpt: "In this post, I provide a detailed walkthrough of the custom CAPTCHA control used on the Technology Toolbox website (including some subtle issues discovered when using this control on Subtext blog pages)."
draft: true
categories: ["Development", "My System"]
tags: ["Subtext", "Web Development"]
---

In case you haven't read the
[introductory post for this series](/blog/jjameson/2011/10/18/introducing-technologytoolbox-com), here is an overview of the CAPTCHA feature
detailed in this post.

> **Note**
>
> Although not illustrated in the screenshots below, the custom CAPTCHA control is also used on the Subtext blog pages to prevent bots from adding comments to blog posts. I'll explain why this is important in a moment.

### Overview

The Technology Toolbox website provides an online form prospective clients
can use regarding potential projects and other contact requests, as shown in
Figure 1.

![Contact form with custom CAPTCHA control](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Contact.png)

    	Figure 1: Contact form with custom CAPTCHA control

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Contact.png)

To ensure responses are generated by a person -- rather than annoying spam
submitted by some worthless "bot" -- the form includes a CAPTCHA control that
requires the correct image to be selected in order to successfully submit the
form. Personally, I prefer this type of challenge-response (i.e. "image-recognition
CAPTCHAs") over the "distorted text" approach that has become common on many
websites today. I often find it frustrating and time consuming to decipher a
random string of characters that have been mashed together along with a wavy
line running through the middle.

When the incorrect image is selected (or no image is selected) in the CAPTCHA
control, an error appears in the validation summary, similar to when required
fields are missing.

![Contact form CAPTCHA validation error](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Contact-CAPTCHA.png)

    	Figure 2: Contact form CAPTCHA validation error

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Contact-CAPTCHA.png)

While the simple image-recognition CAPTCHA is arguably not as "strong" as
a distorted text CAPTCHA (depending, of course, on how much you choose to distort
the text), I think it is sufficient to prevent bots from submitting spam --
at least for now. I can always make it more robust with a little bit of tweaking
if I discover that someone has written code specifically to hack this particular
CAPTCHA implementation.

### s3Capcha jQuery plugin

The foundation for the CAPTCHA control used on TechnologyToolbox.com is based
on a jQuery plugin created by Boban Karišik:

<cite>s3Capcha jQuery plugin</cite>
[http://www.serie3.info/s3capcha/](http://www.serie3.info/s3capcha/)

s3Capcha is simple, yet elegant, and free for commercial use (provided you
agree to preserve the copyright notice in the script file). The only drawback
-- depending on your perspective -- is that Boban's implementation is based
on PHP.

Fortunately, Mahdi Yousefi created an ASP.NET version based on Boban's code:

<cite>Creating an ASP.NET captcha using jQuery and s3capcha</cite>
[http://www.codeproject.com/Articles/38934/Creating-an-ASP-NET-captcha-using-jQuery-and-s3cap](http://www.codeproject.com/Articles/38934/Creating-an-ASP-NET-captcha-using-jQuery-and-s3cap)

I discovered a few issues in Mahdi's code and ended up making some significant
changes for the Technology Toolbox website.

### Refactoring and enhancing the implementation

Note that s3Capcha uses a simple XML file to specify various CAPTCHA options,
such as the images to use and the message instructing the user to select a specific
image. For example, here is the configuration file currently used for TechnologyToolbox.com.

#### config.xml

```
<?xml version="1.0" encoding="utf-8" ?>
<s3capcha>
  <icons>
    <name>apple,cherries,grapes,pear,strawberry,watermelon</name>
    <title>Apple,Cherries,Grapes,Pear,Strawberry,Watermelon</title>
    <width>60</width>
    <height>60</height>
    <ext>jpg</ext>
    <folder>fruit</folder>
  </icons>
  <message>
    To prevent spam from being submitted, please select the following fruit:
    <strong>{0}</strong>
  </message>
</s3capcha>
```

In Mahdi's code, the configuration file is read using the **LoadConfig** method in his **s3capcha** helper class:

```
private static bool LoadConfig()
    {
        string FilePath = "~/s3capcha/config.xml";
        FilePath = HttpContext.Current.Server.MapPath(FilePath);
        if (System.IO.File.Exists(FilePath))
        {
            ...
            return true;
        }
        return false;
    }
```

**LoadConfig** is called from the method used to retrieve the
HTML for the CAPTCHA control:

```
public static string GetHtmlCodes(string PathTo, out int SessionValue)
    {
        bool HasValue = false;
        if (string.IsNullOrEmpty(Message))
            HasValue = LoadConfig();
        else
            HasValue = true;

        if (HasValue)
        {
            ...
            string WriteThis = "<div class=\"s3capcha\"><p>" + ...
            ...
            SessionValue = RandomValues[RandomIndex];
            return WriteThis;
        }
        else
        {
            SessionValue = -1;
            return "Invalid data, config file not found";
        }
    }
```

First of all, I'm not a fan of using return values (`bool`,
`int`, or otherwise) to indicate success/failure (like **LoadConfig** does). I'd much rather see a `void` method that throws an
exception when something goes wrong.

Secondly, there's a subtle bug in Mahdi's code: it is not thread-safe. Note
that **Message** is defined as a `static` member. Consequently
it is *possible* that more than one thread could attempt to initialize
the configuration via the **LoadConfig** method. Is it *probable*?
No, but I still prefer to see it "done right" -- especially in solutions that
I work on.

To encapsulate the CAPTCHA configuration code, I created a new class called
**CaptchaConfiguration**:

```
using System.Collections.ObjectModel;
using System.Globalization;
using System.Web;
using System.Xml;

namespace TechnologyToolbox.Caelum.Website.Controls.Captcha
{
    internal sealed class CaptchaConfiguration
    {
        public const string CaptchaKey = "s3capcha";
                
        public string Message { get; private set; }
        public string IconFolder { get; private set; }
        public string IconFileExtension { get; private set; }

        public int Width { get; private set; }
        public int Height { get; private set; }

        public ReadOnlyCollection<string> IconNames { get; private set; }
        public ReadOnlyCollection<string> IconTitles { get; private set; }

        private static CaptchaConfiguration instance;
        private static readonly object lockObject = new object();

        private CaptchaConfiguration() { }

        public static CaptchaConfiguration Instance
        {
            get
            {
                if (instance == null)
                {
                    lock (lockObject)
                    {
                        if (instance == null)
                        {
                            instance = LoadCaptchaConfiguration();
                        }
                    }
                }

                return instance;
            }
        }

        private static CaptchaConfiguration LoadCaptchaConfiguration()
        {
            CaptchaConfiguration config = new CaptchaConfiguration();

            string configFilename = HttpContext.Current.Server.MapPath(
                "~/Controls/Captcha/config.xml");

            XmlDocument doc = new XmlDocument();
            doc.Load(configFilename);

            string baseNode = "/s3capcha/icons/";

            XmlNode node = doc.SelectSingleNode(baseNode + "name");
            string[] iconNames = node.InnerText.Split(new char[] { ',' });
            config.IconNames = new ReadOnlyCollection<string>(iconNames);

            node = doc.SelectSingleNode(baseNode + "title");
            string[] iconTitles = node.InnerText.Split(new char[] { ',' });
            config.IconTitles = new ReadOnlyCollection<string>(iconTitles);

            node = doc.SelectSingleNode(baseNode + "width");
            config.Width = int.Parse(
                node.InnerText,
                CultureInfo.InvariantCulture);

            node = doc.SelectSingleNode(baseNode + "height");
            config.Height = int.Parse(
                node.InnerText,
                CultureInfo.InvariantCulture);

            node = doc.SelectSingleNode(baseNode + "ext");
            config.IconFileExtension = node.InnerText;

            node = doc.SelectSingleNode(baseNode + "folder");
            config.IconFolder = node.InnerText;

            node = doc.SelectSingleNode("s3capcha/message");
            config.Message = node.InnerXml;

            return config;
        }
    }
}
```

This class uses the
[singleton](http://en.wikipedia.org/wiki/Singleton_pattern) pattern
to ensure that only one instance is ever created. It also ensures thread safety
via the `lock` statement. The **LoadCaptchaConfiguration** method is based on Mahdi's **LoadConfig** method.

Using this class, CAPTCHA configuration paramaters can be accessed using
something like:

```
CaptchaConfiguration.Instance.Message
```

The other issue that I found with Mahdi's implementation is that it doesn't
work with Subtext (at least not in the default configuration). Many of the pages
on the Technology Toolbox website are rendered using the Subtext blog engine
(specifically, any URL under /blog). As I noted at the beginning of this post,
the custom CAPTCHA control is used to prevent spamming via blog post comments.
Also note that Subtext runs with session state and view state disabled by default.

Consequently, if the CAPTCHA control attempts to store the expected value
in session state (as in Mahdi's implementation), an error occurs:

> Session state can only be used when enableSessionState is set to true, either
> in a configuration file or in the Page directive. Please also make sure
> that System.Web.SessionStateModule or a custom session state module is included
> in the &lt;configuration&gt;\&lt;system.web&gt;\&lt;httpModules&gt; section
> in the application configuration.

To resolve this issue, I store the expected value in a cookie instead. Admittedly,
it is not ideal to send the expected value to the client because someone could
crack open the cookie and subsequently craft a hack to submit spam using a "spoofed"
cookie. To mitigate this risk (ever so slightly), I actually store a "super-secret"
hash of the expected value in the cookie.

> **Note**
>
> The reason I put the words "super-secret" in quotes is because someone
> could certainly read this post, figure out the hashing "algorithm" (at
> present, it's almost laughable to use that word), and subsequently create
> a hack to spam the site.
>
> If you do decide to do this, then a) you're a *loser*, and
> b) I'll just quickly change the hashing code to generate some other
> value. In hindsight, I suppose I could have encrypted the expected value
> for the CAPTCHA instead (so that even if you saw the code, you wouldn't
> be able to spoof a valid cookie because you wouldn't know the encryption
> key), but -- well, to be honest, that simply didn't occur to me until
> just this moment.

I also encountered some issues when using the CAPTCHA control on a page where
view state is disabled and an **UpdatePanel** is used to perform
a partial page update (e.g. Subtext blog pages).

One problem was due to a
[known issue with jQuery and update panels](http://stackoverflow.com/questions/256195/jquery-document-ready-and-updatepanels). Fortunately, this one is relatively
easy to fix, once you know what the issue is. However, after fixing that issue,
I soon discovered another problem when using the CAPTCHA control within an
**UpdatePanel** on a blog page (after a comment has been added).
After a little debugging and some subsequent refactoring, I was able to resolve
these issues.

I chose to encapsulate most of the CAPTCHA code from Mahdi's CodeProject
sample in an ASP.NET user control (Captcha.ascx). This is what my original user
control looked like:

```
<%@ Control Language="C#" AutoEventWireup="true" CodeBehind="Captcha.ascx.cs"
    Inherits="TechnologyToolbox.Caelum.Website.Controls.Captcha.CaptchaControl" %>
<script language="javascript" type="text/javascript"
  src="<%=ResolveClientUrl("~/Controls/Captcha/Scripts/s3Capcha.js")%>"></script>
<script language="javascript" type="text/javascript">
    $(document).ready(function () { $('#capcha').s3Capcha(); });
</script>
<div id="capcha">
    <asp:Literal ID="CaptchaPlaceholder" runat="server"></asp:Literal>
    <asp:CustomValidator runat="server" ID="CaptchaValidator"
        CssClass="validator" ErrorMessage="The correct image is not selected."
        Text="(invalid)" OnServerValidate="ValidateCaptchaControl" />
</div>
```

This implementation worked as expected for the Contact form (shown in Figure
1), but not for the Subtext blog pages. After resolving the various issues with
the CAPTCHA control on the blog pages, this is what I ended up with:

#### Captcha.ascx

```
<%@ Control Language="C#" AutoEventWireup="true" CodeBehind="Captcha.ascx.cs"
    Inherits="TechnologyToolbox.Caelum.Website.Controls.Captcha.CaptchaControl" %>
<script language="javascript" type="text/javascript"
  src="<%=ResolveClientUrl("~/Controls/Captcha/Scripts/s3Capcha.js")%>"></script>
<script language="javascript" type="text/javascript">
    $(document).ready(function () { $('#captcha').s3Capcha(); });

    <%--
        If the CAPTCHA is being used on a page where viewstate is disabled and
        an UpdatePanel is used to perform a partial page update, then we need to
        configure the CAPTCHA images again, using the technique described for
        jQuery and ASP.NET UpdatePanels:

        http://stackoverflow.com/questions/256195/jquery-document-ready-and-updatepanels

        Also note that if no UpdatePanel is being used (e.g. on
        /Contact/Default.aspx) then the Sys "namespace" will be undefined.
    --%>
    if (typeof Sys != 'undefined') {
        <%--
            After a comment is submitted for a blog post (using an
            UpdatePanel to perform a partial page update), the "captcha"
            element will no longer be available, so this time use a function
            that checks if the element exists before attempting to configure it
        --%>
        var prm = Sys.WebForms.PageRequestManager.getInstance();
        prm.add_endRequest(configureCaptcha);
    }

    function configureCaptcha() {
        if ($('#captcha') != null) {
            $('#captcha').s3Capcha();
        }
    }
</script>
<div id="captcha">
    <asp:Literal ID="CaptchaPlaceholder" runat="server"></asp:Literal>
    <asp:CustomValidator runat="server" ID="CaptchaValidator"
        CssClass="validator" ErrorMessage="The correct image is not selected."
        Text="(invalid)" OnServerValidate="ValidateCaptchaControl" />
</div>
```

> **Note**
>
> I've kept the comments in the code block above to help readers understand a couple of the more subtle details of the implementation.

#### Captcha.ascx.cs

```
using System;
using System.Collections.Generic;
using System.Globalization;
using System.Text;
using System.Web;
using System.Web.UI.WebControls;

namespace TechnologyToolbox.Caelum.Website.Controls.Captcha
{
    public partial class CaptchaControl : System.Web.UI.UserControl
    {
        private static readonly CaptchaConfiguration config =
            CaptchaConfiguration.Instance;

        public bool IsValid
        {
            get
            {
                bool isValid = false;

                HttpCookie cookie =
                    Request.Cookies[CaptchaConfiguration.CaptchaKey];

                if (cookie != null
                    && string.IsNullOrEmpty(cookie.Value) == false)
                {
                    string formValue =
                        Request.Form[CaptchaConfiguration.CaptchaKey];
                    
                    int numericValue = 0;

                    bool success = int.TryParse(
                        formValue,
                        NumberStyles.Integer,
                        CultureInfo.InvariantCulture,
                        out numericValue);

                    if (success == true)
                    {
                        string hashedFormValue = GetHashedCaptchaValue(
                            numericValue);

                        isValid = (hashedFormValue == cookie.Value);
                    }
                }

                return isValid;
            }
        }

        public string ValidationGroup
        {
            get { return CaptchaValidator.ValidationGroup; }
            set { CaptchaValidator.ValidationGroup = value; }
        }

        private void BindCaptchaControl()
        {
            string basePath = ResolveClientUrl("~/Controls/Captcha");
            
            List<int> values = new List<int>();
            for (int i = 0; i < config.IconNames.Count; i++)
            {
                values.Add(i);
            }

            values = ShuffleList(values);

            Random rand = new Random();
            int randomIndex = rand.Next(0, config.IconNames.Count - 1);

            StringBuilder buffer = new StringBuilder();
            buffer.Append("<div class=\"s3capcha\"><p>");
            buffer.AppendFormat(
                CultureInfo.InvariantCulture,
                config.Message,
                config.IconTitles[values[randomIndex]]);

            buffer.Append("</p>");

            int[] randomValues = new int[config.IconNames.Count];

            for (int i = 0; i < config.IconNames.Count; i++)
            {
                randomValues[i] = rand.Next();

                buffer.AppendFormat(
                    CultureInfo.InvariantCulture,
"<div style=\"float:left\">"
    + "<span>{0} "
        + "<input type=\"radio\" name=\"s3capcha\" value=\"{1}\" />"
    + "</span>"
    + "<div class=\"img\" style=\"background:url({2}) bottom left no-repeat;"
        + " width:{3}px; height:{4}px; cursor:pointer; display:none;\">"
    + "</div>"
+ "</div>",
                    config.IconTitles[values[i]],
                    randomValues[i],
                    basePath + "/icons/" + config.IconFolder + "/"
                        + config.IconNames[values[i]]
                        + "." + config.IconFileExtension,
                    config.Width,
                    config.Height);
            }
            
            buffer.Append("<div style=\"clear:left\"></div></div>");

            CaptchaPlaceholder.Text = buffer.ToString();

            HttpCookie cookie = new HttpCookie(CaptchaConfiguration.CaptchaKey);
            cookie.Value = GetHashedCaptchaValue(randomValues[randomIndex]);
            
            Response.SetCookie(cookie);
        }

        private static string GetHashedCaptchaValue(
            int value)
        {
            string temp = value.ToString(CultureInfo.InvariantCulture);

            int hash = temp.GetHashCode();

            return hash.ToString(CultureInfo.InvariantCulture);
        }

        protected void Page_PreRender(
            object sender,
            EventArgs e)
        {
            if (IsPostBack == false
                || this.Page.EnableViewState == false)
            {
                BindCaptchaControl();
            }
        }

        private static List<int> ShuffleList(
            List<int> input)
        {
            List<int> output = new List<int>();
            Random rnd = new Random();

            while (input.Count > 0)
            {
                int index = rnd.Next(0, input.Count);
                output.Add(input[index]);
                input.RemoveAt(index);
            }

            input.Clear();
            input = null;
            rnd = null;

            return output;
        }
        
        protected void ValidateCaptchaControl(
            object source,
            ServerValidateEventArgs e)
        {
            if (e == null)
            {
                throw new ArgumentNullException("e");
            }

            e.IsValid = IsValid;
        }
    }
}
```

### Subtext bug exposed by the custom CAPTCHA control

While testing my custom CAPTCHA control on a Subtext blog post, I discovered
a bug in the Subtext solution. When the CAPTCHA control failed validation
(either because no image or an incorrect image is selected), the "Submit"
button would show "undefined" after the partial page update.

To fix this bug, I changed the **endRequest** function in
the Subtext common.js file from this:

```
function endRequest(sender, args) {
    //Re-enable button
    var button = $get(sender._postBackSettings.sourceElement.id);
    if (button) {
        button.disabled = false;
        button.value = button.oldValue;
        if (button.className == 'button-disabled') {
            button.className = 'buttonSubmit';
        }
    }
}
```

...to this:

```
function endRequest(sender, args) {
    //Re-enable button
    var button = $get(sender._postBackSettings.sourceElement.id);
    if (button) {
        button.disabled = false;
        if (button.oldValue && button.oldValue != '') {
            button.value = button.oldValue;
        }
        if (button.className == 'button-disabled') {
            button.className = 'buttonSubmit';
        }
    }
}
```
