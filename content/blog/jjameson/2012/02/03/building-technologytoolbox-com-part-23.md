---
title: "Supporting both \"domain.com\" and \"www.domain.com\" (a.k.a. Building TechnologyToolbox.com, part 23)"
date: 2012-02-03T09:38:06+08:00
excerpt: "Can visitors browse your website using \"domain.com\" as well as \"www.domain.com\"? Is this documented in your test spec? It should be -- as I found out...the hard way."
draft: true
categories: ["Development", "My System"]
tags: ["Subtext", "Web Development"]
---

You may have noticed the following code in
[my post from earlier today](building-technologytoolbox-com-part-22.aspx):

```
...
                else if (string.Compare(
                    request.Url.Host,
                    "technologytoolbox.com",
                    StringComparison.OrdinalIgnoreCase) == 0
                    || string.Compare(
                        request.Url.Host,
                        "www.technologytoolbox.com",
                        StringComparison.OrdinalIgnoreCase) == 0)
                {
                    ...
                }
```

I wish I could say this is the way I wrote the code to begin with, but
that wouldn't be honest.

The code above includes a fix for a bug that slipped undetected into
the Production environment. On October 8, 2011, I received an email from
ELMAH due to an unhandled exception on TechnologyToolbox.com:

> System.InvalidOperationException: No analytics key specified for hostname
> (technologytoolbox.com).

Ouch.

If you look at the code in my previous post, you can see where this exception
is thrown. The issue was trivial to fix, but that's not the point.

What is more important is that the site should behave nicely regardless
of whether or not the "www." prefix is specified by a user.

That got me thinking...what would happen if visitors attempted to access
other areas of the site without specifying "www." -- in particular the blog
pages served up by Subtext? After all, in the Subtext **Host Admin**
page, the **Host **field is explicitly set to **www.technologytoolbox.com**
(and, at that point, I had not configured any aliases).

I did end up configuring Subtext to support both www.technologytoolbox.com
and technologytoolbox.com (just to be safe) but I figured I should go one
step further and actually coerce requests for http://technologytoolbox.com
to be redirected to https://www.technologytoolbox.com. This becomes especially
important when using an SSL certificate (for example, when
[automatically redirecting from HTTP to HTTPS for a login form](/blog/jjameson/2009/11/10/sharepoint-web-part-to-redirect-from-http-to-https)).

Fortunately the [URL
Rewrite](http://www.iis.net/download/URLRewrite) module for IIS makes this rather easy. After some quick research,
I used IIS Manager to configure the website to redirect requests for
**technologytoolbox.com/...** to **www.technologytoolbox.com/...**

This resulted in the following elements being added to the Web.config
file:

```
<configuration>
  <system.webServer>
    ...
    <rewrite>
      <rules>
        <rule name="Redirect technologytoolbox.com to www.technologytoolbox.com"
          stopProcessing="true">
          <match url="(.*)" />
          <conditions>
            <add input="{HTTP_HOST}" pattern="^technologytoolbox\.com$" />
          </conditions>
          <action type="Redirect" url="https://www.technologytoolbox.com/{R:1}" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

While doing some background research for this post, I discovered that
a feature was introduced in URL Rewrite 2.0 that provides a similar configuration
but requires far fewer brain cycles than the rule I configured above (not
to mention a fraction of the time to create when using IIS Manager).

Here are the corresponding configuration elements added by the
**Canonical domain name **SEO template in URL Rewrite 2.0:

```
<rewrite>
      <rules>
        <rule name="CanonicalHostNameRule1">
          <match url="(.*)" />
          <conditions>
            <add input="{HTTP_HOST}" pattern="^www\.technologytoolbox\.com$"
              negate="true" />
          </conditions>
          <action type="Redirect" url="https://www.technologytoolbox.com/{R:1}" />
        </rule>
      </rules>
    </rewrite>
```

The rewrite rules are very similar. However there is subtle but important
difference between the two.

The first rule shown above is equivalent to the following:

> For any request in which the host is technologytoolbox.com, redirect
> it to www.technologytoolbox.com.

Whereas the second rule is equivalent to the following:

> For any request in which the host is *not* www.technologytoolbox.com,
> redirect it to www.technologytoolbox.com.

After learning about the new **Canonical domain name **SEO
template, I was tempted to replace the original rule I created back in October.
However, I decided to leave the "handcrafted" rule at this point.

While the behavior would currently be the same regardless of which rule
is used, what if at some point in the future additional hostnames are desired
(e.g. my.technologytoolbox.com)? In that case, the rule added by the SEO
template seems like it could be problematic.

Or maybe I am just missing something? If that is the case, then please...do
tell.

