---
title: "Configuring SSL on SharePoint Sites"
date: 2009-11-09T07:24:00-07:00
excerpt: "If you are using Basic Authentication or Forms-Based Authentication (FBA) with Microsoft Office SharePoint Server (MOSS) 2007 -- or any Web site, for that matter -- then you must configure secure communication (HTTPS) using SSL certificates. 
 However..."
aliases: ["/blog/jjameson/archive/2009/11/08/configuring-ssl-on-sharepoint-sites.aspx", "/blog/jjameson/archive/2009/11/09/configuring-ssl-on-sharepoint-sites.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2009/11/09/configuring-ssl-on-sharepoint-sites.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/11/09/configuring-ssl-on-sharepoint-sites.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that
> blog ever goes away.

If you are using Basic Authentication or Forms-Based Authentication (FBA)
with Microsoft Office SharePoint Server (MOSS) 2007 -- or any Web site, for
that matter -- then you must configure secure communication (HTTPS) using SSL
certificates.

However, you probably didn't select to enable SSL when you originally created
your Web application (after all, you most likely don't want to use Basic Auth
or FBA exclusively, but rather only for users accessing the site externally).
If you followed the process I recommend for customers deploying MOSS solutions,
you created the Web application using an intranet URL (e.g.
[http://fabrikam](http://fabrikam/)) and subsequently
extended the Web application to the **Internet** zone (e.g.
[http://www.fabrikam.com](http://www.fabrikam.com/))
or **Extranet** zone (e.g.
[http://extranet.fabrikam.com](http://extranet.fabrikam.com/))
-- utilizing the SharePoint alternate access mappings feature.

Now that you need to enable secure communication, you might once again consider
extending the Web application -- say to the **Custom** zone --
and select the option to use SSL (e.g.
[https://www.fabrikam.com](https://www.fabrikam.com/)).
After all, this is the recommendation in the following TechNet article:

{{< reference title="Update a Web application URL and IIS bindings (Windows SharePoint Services). (Updated 2009-04-23)." linkHref="http://technet.microsoft.com/en-us/library/cc298636.aspx" >}}

Specifically, here is the text that I am referring to:

{{< blockquote "font-italic" >}}

We do not recommend reusing the same IIS Web site for your HTTP and SSL hosting. Instead, extend a dedicated HTTP and a dedicated SSL Web site, each assigned to its own alternate access mapping zone and URLs.

{{< /blockquote >}}

Wait a minute...let me see if I've got this right...you want me to extend
my Web application -- thereby creating yet another copy of my Web.config files
-- just to enable SSL? That doesn't seem like a good idea.

After all, managing the multitude of Web.config files across multiple Web
application and Web servers in a farm seems to be one of the most challenging
aspects for the Release Management and Operations teams (at least in my experience
during the past several years). [I can't tell you how many times I've compared
Web.config files during the process of troubleshooting various issues and found
discrepancies due to manual changes not getting applied consistently. Sure,
the risk of this is drastically reduced if you leverage the
[`SPWebConfigModification`](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.administration.spwebconfigmodification.aspx) infrastructure, but there are still
rare occasions when a little manual "tweaking" is necessary -- and these often
lead to inconsistent Web.config files.]

Therefore, one of my goals with any SharePoint solution is to minimize the
number of Web.config files that are used. It seems reasonable to expect two
different Web.config files to be used for a site that uses Windows Authentication
internally and Forms Authentication externally, since this is configured using
the `<authentication>`
element.

For example, in the **Default** zone you probably want to specify:

```
<authentication mode="Windows" />
```

Whereas in the **Internet** zone you instead need something
like this:

```
<authentication mode="Forms">
      <forms loginUrl="/Public/Pages/default.aspx" defaultUrl="/" />
    </authentication>
```

Okay, so if we want to avoid creating a new Web application -- and the corresponding
Web.config file(s) -- how can we support both HTTP and HTTPS on the same Web
application? It turns out to be rather simple.

For a starting point, let's assume you've just created your Web application
using the default (i.e. intranet) URL -- say,
[http://fabrikam](http://fabrikam/) -- which uses
Windows Authentication. This is the site that your content authors will use
to manage the site.

The first step is to create an alternate access mapping (AAM).

To configure an alternate access mapping:

1. On the SharePoint Central Administration home page, click the
   **Application Management** tab on the top link bar.
2. On the **Application Management** page, in the **SharePoint Web Application Management** section, click **Create
   or extend Web application**.
3. On the **Create or Extend Web Application** page, in the
   **Adding a SharePoint Web Application** section, click
   **Extend an existing Web application**.
4. On the **Extend Web Application to Another IIS Web Site**page:
   1. In the **Web Application** section, select the Web
      application to extend (e.g.
      [http://fabrikam](http://fabrikam/)).
   2. In the **IIS Web Site** section, in the **Port** and **Host Header** boxes, enter the corresponding
      values such as **80** and **www.fabrikam.com**,
      respectively.
   3. In the **Security Configuration** section, keep the
      default options (you can configure forms authentication, anonymous access,
      and SSL later).
   4. In the **Load Balanced URL** section, ensure the default
      value specified in the **URL** box is correct (e.g.
      **http://www.fabrikam.com:80**) and in **Zone** dropdown list, select **Internet**.
   5. Click **OK**.

The next step is to install your SSL certificate on the site. Once you've
procured your certificate and installed it through Internet Information Services
(IIS) Manager, you then need to add a public URL in SharePoint for HTTPS and
add an HTTPS binding in IIS.

To add a public URL to HTTPS:

1. On the SharePoint Central Administration home page, click the
   **Operations** tab on the top link bar.
2. On the **Operations** page, in the **Global Configuration** section, click **Alternate access mappings**.
3. On the **Alternate Access Mappings** page, click
   **Edit Public URLs**.
4. On the **Edit Public Zone URLs**page:
   1. In the **Alternate Access Mapping Collection** section,
      select the Web application (e.g.
      [http://fabrikam](http://fabrikam/)).
   2. In the **Public URLs** section, copy the URL from the
      **Internet** box to the **Custom** box, and
      change **http://** to **https://**.
   3. Click **Save**.

To add an HTTPS binding to the site in IIS:

1. Click **Start**, point to **Administrative Tools**,
   and then click **Internet Information Services (IIS) Manager**.
2. In Internet Information Services (IIS) Manager, click the plus sign
   (+) next to the server name that contains the Web application, and then
   click the plus sign next to **Sites** to view the Web applications
   that have been created.
3. Click the name of the Web application corresponding to the **Internet**
   zone (e.g. **SharePoint - www.fabrikam.com80**). In the
   **Actions** section, under the **Edit Site** heading,
   click **Bindings...**.
4. In the **Site Bindings** window, click **Add**.
5. In the **Add Site Binding**window:
   1. In the **Type:** dropdown, select **https**.
   2. In the **SSL Certificate:** dropdown, select the certificate
      corresponding to the site (e.g. www.fabrikam.com).
   3. Click **OK**.
   4. In the **Site Bindings** window, click **Close**.

At this point, your SharePoint site supports Windows Authentication both
internally (via [http://fabrikam](http://fabrikam/))
and externally (via [http://www.fabrikam.com](http://www.fabrikam.com/)
and [https://www.fabrikam.com](https://www.fabrikam.com/)).
The final steps are to enable anonymous access and configure Forms Authentication.

Note that after configuring Forms Authentication, users will not automatically
be redirected from HTTP to HTTPS for the login page. To achieve this user experience,
I've used a couple of techniques in the past that are based on the same fundmental
concept. The first technique utilized code on the login page to automatically
detect HTTP connections and redirect to HTTPS. The second technique was essentially
the same logic, but rather than utilizing code embedded in the login page, it
is encapsulated in a Login Form Web Part. I'll discuss this redirect code in
a follow-up post.

