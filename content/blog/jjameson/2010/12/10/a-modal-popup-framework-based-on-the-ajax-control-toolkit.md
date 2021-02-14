---
title: "A Modal Popup Framework Based on the AJAX Control Toolkit"
date: 2010-12-10T02:40:00+08:00
excerpt: "The \"Announcements\" feature that I developed for a customer about a year ago uses a modal popup window to display content to users. The solution leverages the AJAX Control Toolkit to render the modal popups &ndash; specifically the ModalPopupExtender..."
draft: true
categories: ["Development"]
tags: ["Web Development"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/12/10/a-modal-popup-framework-based-on-the-ajax-control-toolkit.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/12/10/a-modal-popup-framework-based-on-the-ajax-control-toolkit.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

The "Announcements" feature that I developed for a customer about a year ago uses         a modal popup window to display content to users. The solution leverages the [AJAX Control Toolkit](http://www.asp.net/AJAX/AjaxControlToolkit/Samples/Default.aspx) to render the modal popups -- specifically the **ModalPopupExtender**         class.

Refer to the following site for more information on the **ModalPopupExtender**         class, including a sample page that demonstrates a popup window:

<cite>ModalPopup Demonstration</cite>
[http://www.asp.net/AJAX/AjaxControlToolkit/Samples/ModalPopup/ModalPopup.aspx](http://www.asp.net/AJAX/AjaxControlToolkit/Samples/ModalPopup/ModalPopup.aspx)

To encapsulate the functionality for rendering modal popup windows, the "Announcements"         feature builds upon a lightweight "Modal Popup Framework." [Admittedly, calling         this a "framework" is a little bit of a stretch.]

The framework essentially consists of the **ModalPopupWebPart** abstract         base class which is primarily responsible for rendering the **ModalPanel**         (i.e. a `<div>` element that serves as the container for content         to display in the modal popup window), the **Title Bar**, and link         buttons (**OK** and **Cancel**) to dismiss the popup window.

> **Note**
> 
>             Web Parts that derive from **ModalPopupWebPart** are responsible for
>             displaying the main content of the popup window. This is typically done by overriding
>             the [CreateChildControls](http://msdn.microsoft.com/en-us/library/system.web.ui.control.createchildcontrols.aspx) method to add controls to the **ModalPanel**.

The following figure illustrates the various UI elements of a modal popup window.

![Elements of a modal popup window](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Modal-Popup-Window-Elements.png)
Figure 1: Elements of a modal popup window

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Modal-Popup-Window-Elements.png)

As noted previously, the **ModalPopupWebPart** base class does not         specify the main content to render in the popup window; nor does it specify the         dimensions (width and height) of the popup window. Instead, Web Parts that derive         from **ModalPopupWebPart** specify the content (and, optionally, the         width and height of the main content).

For example, the modal popup window shown in Figure 1 was generated using the following         sample code:

```
using System;
using System.Web.UI;
using System.Web.UI.WebControls;

namespace Fabrikam.Demo.Web.UI.WebControls
{
    public class SamplePopupWebPart : ModalPopupWebPart
    {
        private Panel modalPanelContent;
        private LiteralControl content;

        protected override void CreateChildControls()
        {
            base.CreateChildControls();

            modalPanelContent = new Panel();
            modalPanelContent.CssClass = "modalPanelContent";
            modalPanelContent.Width = new Unit(400);
            base.ModalPanel.Controls.Add(modalPanelContent);
            
            content = new LiteralControl();
            modalPanelContent.Controls.Add(content);
            content.Text =
@"<div class='mainContent'>Lorem ipsum dolor sit amet, consectetur adipiscing
 elit. Cras imperdiet ligula ut turpis tristique ultricies. Aliquam accumsan mi
 at ligula porta consectetur. Maecenas aliquam dictum dui quis venenatis. Donec
 eros velit, dictum vel pretium eu, tempus eu neque. Aliquam tortor mi, lacinia
 sit amet iaculis sit amet, porttitor sed neque. Sed eu turpis ante. Sed
 consequat suscipit elit, non posuere sapien hendrerit pharetra. Integer dapibus
 tortor a magna convallis congue. Mauris mollis turpis eu enim ornare
 sollicitudin. Etiam nec lacus libero. Aliquam mattis rhoncus bibendum. Nunc
 placerat nibh ut odio rhoncus ut gravida est suscipit. Etiam nec massa ut nibh
 mattis fermentum rutrum vel tortor. Phasellus vitae eros ante.</div>";

        }
    }
}
```

Note that by specifying the height of the **modalPanelContent** element:

```
modalPanelContent.Height = new Unit(100);
```

&hellip;and specifying a corresponding CSS rule:

```
.modalPanel div.modalPanelContent {
    overflow: auto;
}
```

&hellip;it is possible to constrain the height of the modal popup window and         show a scrollbar as necessary. This is illustrated in the following figure.

![Constraining the height of content displayed in a modal popup window](https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Modal-Popup-Window-Constrained-Height.png)
Figure 2: Constraining the height of content displayed in a modal popup window

In addition to creating various user interface elements for modal popup windows,         the **ModalPopupWebPart** also converts client-side events (e.g. clicking         the **OK** link button) into server-side events. Note that this postback         occurs asynchronously in order to preserve the "AJAX" experience by avoiding a refresh         of the entire page. The **ModalPopupWebPart** base class also allows         the labels of the **OK** and **Cancel** link buttons to         be changed by derived classes, as well as the ability to hide either of the link         buttons.

For example, it might be desireable to hide the **Cancel** link button         and change the text of the **OK** button to be "Close" instead. When         the user clicks the "Close" button, the derived Web Part can specify a server-side         event handler (or simply override the **OnModalPopupOk** method) to         perform some action.

```
protected override void OnModalPopupOk(
            EventArgs e)
        {
            Debug.WriteLine("SamplePopupWebPart::OnModalPopupOk()");

            base.OnModalPopupOk(e);

            // Do something here...
        }
```

If you would like to see the "Modal Popup Framework" in action, simply download         and unzip the attachment to this post, open the Visual Studio 2010 solution, and         then run it.

In my next post, I'll describe the "Announcements" feature that builds upon this         framework in order to show marketing announcements on a SharePoint site.

