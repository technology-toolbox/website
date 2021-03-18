---
title: AJAX in MOSS 2007 -- The Easy Way, Part 2
date: 2010-03-24T04:36:00-06:00
excerpt:
  "In my previous post , I showed how you can quickly create an AJAX-enabled Web
  application in Microsoft Office SharePoint Server (MOSS) 2007. I also provided
  a sample AJAX Web Part, illustrated in the following screenshot: Figure 1:
  AJAX in SharePoint..."
aliases:
  [
    "/blog/jjameson/archive/2010/03/23/ajax-in-moss-2007-the-easy-way-part-2.aspx",
    "/blog/jjameson/archive/2010/03/24/ajax-in-moss-2007-the-easy-way-part-2.aspx",
  ]
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/03/24/ajax-in-moss-2007-the-easy-way-part-2.aspx"
---

In my
[previous post](/blog/jjameson/2010/03/23/ajax-in-moss-2007-the-easy-way-part-1),
I showed how you can quickly create an AJAX-enabled Web application in Microsoft
Office SharePoint Server (MOSS) 2007.

I also provided a sample AJAX Web Part, illustrated in the following screenshot:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/AJAX-in-SharePoint-600x417.png"
alt="AJAX in SharePoint" class="screenshot" height="417" width="600"
title="Figure 1: AJAX in SharePoint" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/SharePoint/AJAX-in-SharePoint-989x688.png)

Assuming you are familiar with the
**[UpdatePanel](http://msdn.microsoft.com/en-us/library/system.web.ui.updatepanel.aspx)**
class in ASP.NET AJAX, the sample Web Part should be mostly self-explanatory:

```
using System;
using System.Web.UI;
using System.Web.UI.WebControls;
using System.Web.UI.WebControls.WebParts;

namespace Fabrikam.Demo.Web.WebParts
{
    /// <summary>
    /// A sample Web Part used to demonstrate AJAX update functionality.
    /// </summary>
    public class SampleAjaxUpdateWebPart : AjaxWebPart
    {
        Label timestamp;

        /// <summary>
        /// Called by the ASP.NET page framework to notify server controls that
        /// use composition-based implementation to create any child controls
        /// they contain in preparation for posting back or rendering.
        /// </summary>
        protected override void CreateChildControls()
        {
            base.CreateChildControls();

            LiteralControl literal = new LiteralControl(
                "<p>Web Part loaded at: " + DateTime.Now + "</p>");

            this.Controls.Add(literal);

            UpdatePanel updatePanel = new UpdatePanel();
            updatePanel.ID = "updatePanel";
            updatePanel.UpdateMode = UpdatePanelUpdateMode.Conditional;
            this.Controls.Add(updatePanel);

            timestamp = new Label();
            timestamp.ID = "timestamp";
            timestamp.Text = "Click <b>Update</b> to update this label";
            updatePanel.ContentTemplateContainer.Controls.Add(timestamp);

            Panel buttonPanel = new Panel();
            this.Controls.Add(buttonPanel);

            LinkButton updateButton = new LinkButton();
            updateButton.ID = "update";
            updateButton.Text = "Update";
            updateButton.Click += new EventHandler(updateButton_Click);
            buttonPanel.Controls.Add(updateButton);

            AsyncPostBackTrigger updateButtonPostBackTrigger =
                new AsyncPostBackTrigger();

            updateButtonPostBackTrigger.ControlID = updateButton.ID;
            updatePanel.Triggers.Add(updateButtonPostBackTrigger);
        }

        private void updateButton_Click(
            object sender,
            EventArgs e)
        {
            timestamp.Text = "Timestamp updated at: " + DateTime.Now;
        }
    }
}
```

However, notice that the sample Web Part inherits from my custom **AjaxWebPart**
class. Before looking at the details of the **AjaxWebPart** class, let's see
what happens when **SampleAjaxUpdateWebPart** is changed to inherit from
**System.Web.UI.WebControls.WebParts.WebPart** instead.

After changing the base class and building the solution, the following commands
are used to update the assembly in the GAC and recycle the application pool for
the Fabrikam site:

```
cd \NotBackedUp\Fabrikam\Demo\Main\Source\Web\DeploymentFiles\Scripts
"GAC Assemblies.cmd"
C:\Windows\System32\inetsrv\appcmd.exe recycle apppool "SharePoint - fabrikam-local80"
```

Attempting to browse to the home page of the site now results in an error. After
tweaking the Web.config file to set `<SafeMode CallStack="true" ...>` and
`<customErrors mode="Off" />`, the details of the error are revealed:

{{< blockquote "font-italic text-danger" >}}

System.InvalidOperationException: The control with ID 'updatePanel' requires a
ScriptManager on the page. The ScriptManager must appear before any controls
that need it.

{{< /blockquote >}}

This is no real surprise, since BlueBand.master doesn't declare an instance of
the ASP.NET
**[ScriptManager](http://msdn.microsoft.com/en-us/library/system.web.ui.scriptmanager.aspx)**
(which is required for providing the ASP.NET AJAX script files on any
AJAX-enabled page). Mike Ammerlaan covers this in
[his original post](http://sharepoint.microsoft.com/blogs/mike/Lists/Posts/Post.aspx?ID=3)
that I referenced in yesterday's post.

While we *could* modify BlueBand.master to declare a **ScriptManager**, an
alternative is to instead use a little bit of code in the
**CreateChildControls** method of the Web Part to dynamically create one, if
necessary:

```
            if (ScriptManager.GetCurrent(this.Page) == null)
            {
                ScriptManager scriptHandler = new ScriptManager();
                scriptHandler.ID = "scriptHandler";
                scriptHandler.EnablePartialRendering = true;
                this.Controls.Add(scriptHandler);
            }
```

Mike's post also describes adding the following startup script in order to
enable UpdatePanels to be used on a page:

```
<script type='text/javascript'>
  _spOriginalFormAction = document.forms[0].action;
  _spSuppressFormOnSubmitWrapper=true;
</script>
```

In my experience, the UpdatePanels seem to work just fine without this startup
script, but there are scenarios where out-of-the-box SharePoint functionality is
broken after enabling AJAX if you don't add this startup script. For example,
the **Edit Page** button in the page editing toolbar sometimes stops working. It
doesn't seem to happen all the time, but when it does, it's obviously very
frustrating.

I've also seen the following error when using the
**[ModalPopupExtender](http://www.asp.net/AJAX/AjaxControlToolkit/Samples/ModalPopup/ModalPopup.aspx)**
from the AJAX Control Toolkit on a SharePoint site:

{{< blockquote "font-italic text-danger" >}}

Extender Controls may not be Registered before PreRender

{{< /blockquote >}}

To avoid this error, you have to force the child controls to be created during
the Init phase of the page lifecycle.

The custom **AjaxWebPart** base class handles all of the fixup necessary to
avoid these issues:

```
using System;
using System.Web.UI;
using System.Web.UI.WebControls.WebParts;

using Microsoft.SharePoint.Utilities;

namespace Fabrikam.Demo.Web.WebParts
{
    /// <summary>
    /// Serves as the base class for custom ASP.NET Web Part controls that
    /// utilize AJAX for asynchronous partial page updates.
    /// </summary>
    public class AjaxWebPart : BaseWebPart
    {
        /// <summary>
        /// Raises the <see cref="System.Web.UI.Control.Init"/> event.
        /// </summary>
        /// <remarks>
        /// This method is overridden to mitigate known issues with AJAX and
        /// SharePoint.
        /// </remarks>
        /// <param name="e">An <see cref="System.EventArgs"/> object that
        /// contains the event data.</param>
        protected override void OnInit(
            EventArgs e)
        {
            base.OnInit(e);

            // HACK: In SharePoint, EnsureChildControls() must be called during
            // the Init phase of the page life cycle in order to avoid an error:
            //
            // "Extender Controls may not be Registered before PreRender"
            EnsureChildControls();

            // HACK: AJAX breaks the OOTB functionality in SharePoint (e.g. the
            // "Edit Page" button)
            //
            // http://sharepoint.microsoft.com/blogs/mike/Lists/Posts/Post.aspx?ID=3
            ScriptManager.RegisterStartupScript(
                this,
                typeof(AjaxWebPart),
                "UpdatePanelFixup",
                "_spOriginalFormAction = document.forms[0].action;"
                    + " _spSuppressFormOnSubmitWrapper=true;",
                true);
        }

        /// <summary>
        /// Called by the ASP.NET page framework to notify server controls
        /// that use composition-based implementation to create any child
        /// controls they contain in preparation for posting back or rendering.
        /// </summary>
        /// <remarks>This method is overridden to ensure the page contains an
        /// instance of the <see cref="System.Web.UI.ScriptManager"/> class.
        /// </remarks>
        protected override void CreateChildControls()
        {
            base.CreateChildControls();

            if (ScriptManager.GetCurrent(this.Page) == null)
            {
                ScriptManager scriptHandler = new ScriptManager();
                scriptHandler.ID = "scriptHandler";
                scriptHandler.EnablePartialRendering = true;
                this.Controls.Add(scriptHandler);
            }
        }
    }
}
```

The **BaseWebPart** class simply provides the ability to render an error message
instead of whatever content would normally be rendered in the Web Part if an
error had not occurred. It's only purpose in this case is to enable Web Parts
that derive from **AjaxWebPart** to use the error handling feature.

```
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls.WebParts;

namespace Fabrikam.Demo.Web.WebParts
{
    /// <summary>
    /// Serves as the base class for custom ASP.NET Web Part controls, adding
    /// basic error handling functionality.
    /// </summary>
    public abstract class BaseWebPart : WebPart
    {
        /// <summary>
        /// Gets or sets the message to display when an error occurs with the
        /// Web Part.
        /// </summary>
        protected string ErrorMessage { get; set; }

        /// <summary>
        /// Renders the contents of the control to the specified writer.
        /// </summary>
        /// <remarks>
        /// This method is overridden to determine whether to render the child
        /// controls or an error message.</remarks>
        /// <param name="writer">The <c>HtmlTextWriter</c> object that receives
        /// the server control content.</param>
        protected override void RenderContents(
            HtmlTextWriter writer)
        {
            if (string.IsNullOrEmpty(this.ErrorMessage) == true)
            {
                base.RenderContents(writer);
            }
            else
            {
                writer.AddAttribute(
                    HtmlTextWriterAttribute.Class,
                    "errorMessage");

                writer.RenderBeginTag(HtmlTextWriterTag.P);
                HttpUtility.HtmlEncode(this.ErrorMessage, writer);
                writer.RenderEndTag();
            }
        }
    }
}
```

In a future post, I'll describe the AJAX modal popup framework that I developed
for MOSS 2007 (for example, to display announcements on a site).
