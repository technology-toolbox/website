---
title: "DataNavigateUrlFormatString Does Not Allow \"javascript:\""
date: 2009-10-26T00:49:00-07:00
excerpt: "I encountered an interesting bug last Friday with the ASP.NET HyperLinkField control. To understand the scenario, think of the typical \"view detail\" feature when showing summary data in a table. In other words, you want to provide users the ability to..."
draft: true
categories: ["Development"]
tags: ["Web Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/26/datanavigateurlformatstring-does-not-allow-javascript.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/26/datanavigateurlformatstring-does-not-allow-javascript.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

I encountered an interesting bug last Friday with the ASP.NET [HyperLinkField](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.hyperlinkfield.aspx) control. To understand the scenario, think of the typical "view detail" feature when showing summary data in a table. In other words, you want to provide users the ability to click a cell in a table (e.g. a [GridView](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.gridview.aspx) control) in order to see more information.

I've implemented this scenario before, so I was a stumped when I couldn't get it to work right away. However, this time the scenario was a little different because we want to display a popup window to an external site. Also note that several visual properties of the popup window need to be specified for our application -- such as the width and height of the window, as well as hiding the menu bar, status bar, etc.

In other words, we need the links in the table to actually call a JavaScript function instead of specifying a simple URL.

My initial attempt was to specify something like :

```
<asp:GridView ID="summaryGrid" runat="server"
    AutoGenerateColumns="False"
    ...>
    ...
    <Columns>        
        <asp:HyperLinkField DataNavigateUrlFields="Id"
           DataNavigateUrlFormatString="javascript:ShowItemDetail({0})"
           DataTextField="CreateTime"
           DataTextFormatString="{0:ddd MMM dd HH:mm}"
           HeaderStyle-CssClass="dateColumn"
           HeaderText="Date" />  
       ...
    </Columns>
</asp:GridView>
```

Note that `ShowItemDetail` is a JavaScript function that simply calls `window.open` (specifying the URL as well as additional "feature" parameters such as the window width and height).

Unfortunately, if you specify something like `"javascript:ShowItemDetail({0});"` in [DataNavigateUrlFormatString](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.hyperlinkfield.datanavigateurlformatstring.aspx), you will find that ASP.NET renders an anchor element with no `href` attribute (e.g. `<a>Mon Oct 26 08:23</a>`) -- which obviously doesn't work. This appears to be "by design" -- in order to "mitigate against cross-site scripting attacks."

Refer to the following for more detail:

{{< reference title="asp:GridView does not render HyperLinkField when a colon (:) is included in the DataNavigateUrlFormatString. 2007-06-05." linkHref="https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=102300" >}}

To workaround this, use a [TemplateField](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.templatefield.aspx) instead:

```
<asp:GridView ID="summaryGrid" runat="server"
    AutoGenerateColumns="False"
    ...>
    ...
    <Columns>
        <%-- HACK: Ideally, we would just use an asp:HyperLinkField,
        but there's a bug when specifying "javascript:" in the
        DataNavigateUrlFormatString:

        https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=102300

        As a workaround, use a TemplateField instead. --%>
        <asp:TemplateField>
           <HeaderTemplate>Date</HeaderTemplate>
           <HeaderStyle CssClass="dateColumn" />
           <ItemTemplate>
               <a href='javascript:ShowItemDetail(<%# Eval("Id")%>)'>
                   <%# Eval("CreateTime", "{0:ddd MMM dd HH:mm}") %>
               </a>
           </ItemTemplate>
        </asp:TemplateField>
        ...
    </Columns>
</asp:GridView>
```

Or, if you are really paranoid about the performance impact of using reflection in the DataBinder.Eval method, you can use strongly typed objects instead:

```
<asp:GridView ID="summaryGrid" runat="server"
   AutoGenerateColumns="False"
   ...>
   ...
   <Columns>
      <%-- HACK: Ideally, we would just use an asp:HyperLinkField,
      but there's a bug when specifying "javascript:" in the
      DataNavigateUrlFormatString:
        
      https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=102300
        
      As a workaround, use a TemplateField instead.
      --%>
      <asp:TemplateField>
         <HeaderTemplate>Date</HeaderTemplate>
         <HeaderStyle CssClass="dateColumn" />
         <ItemTemplate>
             <a href='javascript:ShowItemDetail(<%# ((Fabrikam.Demo.Integration.Incident)Container.DataItem).Id %>)'>
                 <%# ((Fabrikam.Demo.Integration.Incident)Container.DataItem).CreateTime.ToString("ddd MMM dd HH:mm") %>
             </a>
         </ItemTemplate>
      </asp:TemplateField>
      ...
   </Columns>
</asp:GridView>
```

Of course, if you choose to go the strongly-typed route, you will need to reference the assembly containing your custom business objects. For example:

```
<%@ Assembly Name="Fabrikam.Demo.Integration, Version=1.0.0.0, Culture=neutral, PublicKeyToken=786f58ca4a6e3f60" %>
```

