---
title: "Filter ELMAH email messages to avoid getting spammed by hackers"
date: 2012-02-28T13:24:53-07:00
excerpt: "I finally got around to configuring an ELMAH filter for the TechnologyToolbox.com website (so I wouldn't be bothered by frequent email messages due to failed hack attempts). During the process, I also discovered a couple of bugs in ELMAH (and learned a lot more about the internal workings of ELMAH)."
aliases: ["/blog/jjameson/archive/2012/02/28/filter-elmah-email-messages-to-avoid-getting-spammed-by-hackers.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["Web Development"]
---

As I described in
[a previous post](/blog/jjameson/2012/01/22/building-technologytoolbox-com-part-14), TechnologyToolbox.com currently uses ELMAH to log errors
that occur on the website. In
[a follow-up post](/blog/jjameson/2012/01/23/building-technologytoolbox-com-part-15) I also discussed some of the errors that have occurred
since the website went live last year -- many of which result from failed attempts
by hackers.

Last Friday I received 23 email messages from ELMAH because the scoundrel
with IP address

96.31.39.146 apparently had nothing better to do with his or her time than
try to
manipulate the ASP.NET view state (apparently via some automated tool, based
on the number of errors in such as short amount of time).

Even though
I previously configured
[Outlook rules](/blog/jjameson/2010/01/04/managing-email-effectively) to automatically move these messages out of my inbox, I was
tired of seeing these messages altogether. So this past weekend, I decided
to add a filter to ELMAH to avoid sending messages when an error occurs as a
result of a known hack attempt.

> **Note**
>
> TechnologyToolbox.com is configured to log errors to both email and
> SQL Server. I still believe it is important to track all errors on the
> site (for example, to keep tabs on the frequency of hack attempts).
> However I'm okay with not getting notified via email whenever, for example,
> someone attempts to hack the ASP.NET view state.

It turned out to be a very educational experience for me because I ended
up diving into the ELMAH source code in order to investigate some issues. I
even discovered a couple of bugs in ELMAH that apparently have gone unnoticed
up to this point. More on those issues in a moment...

Let's start with a simple scenario -- or, more precisely, a very basic hack
attempt. Specifically, imagine for a moment that you are a hacker and you want
to "probe" an ASP.NET website to see if it is configured to block potentially
malicious input strings submitted by the client. In other words, you want to
see if the **validateRequest** attribute has been set to
**false** in the Web.config file or on a specific page. [If your
website currently does this, I really hope you know what you are doing.]

You can quickly check this by specifying a "?&lt;script&gt;" query string
when requesting a page. Assuming the default (i.e. secure) configuration, an
**HttpRequestValidationException** will occur, and -- assuming
the ELMAH **ErrorMailModule** is configured -- the exception will
trigger an email to the designated recipients.

Since, like me, you probably don't want to be bothered by a message whenever
someone attempts this hack, you could configure an ELMAH filter as follows:

```
<elmah>
    ...
    <errorFilter>
      <test>
        <is-type binding="BaseException"
          type="System.Web.HttpRequestValidationException, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
      </test>
    </errorFilter>
  </elmah>
```

However, there's a problem with the configuration shown above: it completely
filters errors resulting from **HttpRequestValidationException**.
Recall what I said earlier about still wanting to log the errors to SQL Server,
while preventing "spam" messages from being sent to email recipients.

Consequently we need to tweak the error filter a little bit (by adding an
`<and>`
element and checking if the current "filter source" is the **ErrorMailModule**):

```
<errorFilter>
      <test>
        <!-- Do not send email notification when hackers attempt something like "http://.../?<script>" -->
        <and>
          <equal binding="FilterSourceType.Name" value="ErrorMailModule"
            type="String" />
          <is-type binding="BaseException"
            type="System.Web.HttpRequestValidationException, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
        </and>
      </test>
    </errorFilter>
```

With the configuration shown above, anytime someone causes an **HttpRequestValidationException**,
the error is logged via **ErrorLogModule** (which, in my configuration
means it is written to the SQL Server database) but the email message is suppressed,
courtesy of the filter.

Another common hack attempt (according to the errors I've seen on TechnologyToolbox.com)
is to attempt to hack the ASP.NET view state. In these scenarios, the email
messages sent by ELMAH typically specify one of the following:

- System.FormatException: Invalid length
  for a Base-64 char array.
- System.FormatException: Invalid character
  in a Base-64 string.

These are actually the "inner" exceptions. If you look at the details for
the error, you'll see the following:

{{< blockquote "font-italic text-danger" >}}

System.Web.HttpException: The state information is invalid for this page and might be corrupted. ---&gt; System.Web.UI.ViewStateException: Invalid viewstate.

{{< /blockquote >}}

To suppress these messages, we can simply expand the ELMAH filter a little
bit:

```
<errorFilter>
      <test>
        <or>
          <!-- Do not send email notification when hackers attempt something like "http://.../?<script>" -->
          <and>
            <equal binding="FilterSourceType.Name" value="ErrorMailModule"
              type="String" />
            <is-type binding="BaseException"
              type="System.Web.HttpRequestValidationException, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
          </and>
          <!-- Do not send email notification when someone attempts to hack view state -->
          <and>
            <equal binding="FilterSourceType.Name" value="ErrorMailModule"
              type="String" />
            <equal binding="Exception.Message"
              value="The state information is invalid for this page and might be corrupted." type="String" />
          </and>
        </or>
      </test>
    </errorFilter>
```

Another common hack that I've seen attempted somewhat frequently against
TechnologyToolbox.com is the so-called "padding oracle exploit" identified back
in 2010 ([http://technet.microsoft.com/en-us/security/advisory/2416728](http://technet.microsoft.com/en-us/security/advisory/2416728)).
The following errors are typically indicative of these hack attempts:

- System.Web.HttpException: This is an
  invalid script resource request.
- System.Web.HttpException: This is an
  invalid webresource request.

Consequently we might as well add these to the ELMAH filter:

```
<errorFilter>
      <test>
        <or>
          <!-- Do not send email notification when hackers attempt something
            like "http://.../?<script>" -->
          <and>
            <equal binding="FilterSourceType.Name" value="ErrorMailModule"
              type="String" />
            <is-type binding="BaseException"
              type="System.Web.HttpRequestValidationException, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
          </and>
          <!-- Do not send email notification when someone attempts to hack view
            state -->
          <and>
            <equal binding="FilterSourceType.Name" value="ErrorMailModule"
              type="String" />
            <equal binding="Exception.Message"
              value="The state information is invalid for this page and might be corrupted."
              type="String" />
          </and>
          <!-- Do not send email notification when someone attempts to hack the
            site using the "padding oracle exploit"
            (http://technet.microsoft.com/en-us/security/advisory/2416728) -->
          <and>
            <equal binding="FilterSourceType.Name" value="ErrorMailModule"
              type="String" />
            <or>
              <equal binding="Exception.Message"
                value="This is an invalid script resource request."
                type="String" />
              <equal binding="Exception.Message"
                value="This is an invalid webresource request."
                type="String" />
            </or>
          </and>
        </or>
      </test>
    </errorFilter>
```

Obviously you could choose to refactor the filter configuration shown above
a little bit (to eliminate the duplicate checks on `FilterSourceType.Name`. However there's
a bigger issue that I discovered when testing the filter: it works fine when
running in Full trust, but does not work in Medium trust.

I created ELMAH issue 277 to track this bug:

{{< reference title="errorFilter does not work in Medium trust configuration (SecurityException - \"Request for the permission of type 'System.Security.Permissions.ReflectionPermission...' failed\")" linkHref="http://code.google.com/p/elmah/issues/detail?id=277" >}}

You can read the details by following the link above, if you are interested,
but here is the gist of it:

> The filter should work in Medium trust (meaning the error should be logged,
> but an email should not be sent from ELMAH due to the filter). Instead of
> correctly applying the filter in the Medium trust configuration, a
> **SecurityException** is thrown and the error is not logged
> at all (even by the **ErrorLogModule**).

Ouch.

James Driscoll pointed out that you can avoid the issue in some cases, for
example, by changing:

```
<equal binding="FilterSourceType.Name" value="ErrorMailModule"
    type="String" />
```

to:

```
<is-type binding="FilterSource" type="Elmah.ErrorMailModule" />
```

However, that may not be viable for all scenarios (such as comparing the
exception message to a string).

I noticed today that Atif added a comment saying this is "by-design" and
marked the issue as "invalid."

Personally I'd much rather see the status designate something like
**Won't Fix** (like we used to say back when I worked at Microsoft)
rather than **Invalid** -- but I suppose this is more of a nitpick
with the setup of [http://code.google.com](http://code.google.com).

"Invalid" makes me think this isn't a bug -- and at the very least, I would
really like to see some **Important** note on the
[ELMAH error filter
page](http://code.google.com/p/elmah/wiki/ErrorFiltering) stating the known caveat when running in Medium trust.

Issue 277 became somewhat of a moot point last weekend because once I discovered
the bug, I quickly turned my attention to finding a workaround. Consequently,
I converted my filter configuration to use JavaScript instead:

```
<errorFilter>
      <test>
        <jscript>
          <expression>
            <![CDATA[
              // @assembly mscorlib
              // @assembly System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a
              // @import System.Web

              // Do not send email notification when hackers attempt something
              // like "http://.../?<script>"
              (FilterSourceType.Name == "ErrorMailModule"
                && BaseException instanceof HttpRequestValidationException)
              // Do not send email notification when someone attempts to hack
              // view state
              || (FilterSourceType.Name == "ErrorMailModule"
                && Exception.Message ==
                  "The state information is invalid for this page and might be corrupted.")
              // Do not send email notification when someone attempts to hack
              // the site using the "padding oracle exploit"
              // (http://technet.microsoft.com/en-us/security/advisory/2416728)
              || (FilterSourceType.Name == "ErrorMailModule"
                && (Exception.Message ==
                    "This is an invalid script resource request."
                  || Exception.Message ==
                    "This is an invalid webresource request."))
              ]]>
          </expression>
        </jscript>
      </test>
    </errorFilter>
```

However, while initially testing this new approach, I found that I still
received email messages that should have been blocked by the filter. Thinking
that I must have translated my filter incorrectly (from the various XML elements
to the JScript expression), I took a little detour and spent about a half hour
creating some unit tests for ELMAH (which I will share in
[a separate post](/blog/jjameson/2012/02/29/unit-tests-for-filtering-errors-in-elmah)).

The problem, as it turns out, was due to the fact that I ran my initial tests
with the JavaScript filter under Full trust. When I ran the tests in Medium
trust, the JavaScript filter worked as expected.

In other words, I discovered another issue that only occurred in Full trust
-- not Medium trust.

I created ELMAH issue 278 to track this bug:

{{< reference title="JScript error filter does not work in Full trust configuration (when using FilterSourceType.Name)" linkHref="http://code.google.com/p/elmah/issues/detail?id=278" >}}

You can read the details by following the link above, if you are interested,
but here is the gist of it:

> It appears there is some sort of caching bug in **FullTrustEvaluationStrategy**
> when using **FilterSourceType.Name** (in other words,
> **FilterSourceType.Name** is not evaluated the second time
> through when the **ErrorMailModule** context object is passed
> to the **Eval** method).
>
> The filter should work the same in Full trust and Medium trust
> (meaning the error should be logged, but an email should not be sent from
> ELMAH due to the filter). Instead of correctly applying the filter in the
> Full trust configuration, **FilterSourceType.Name** is not
> evaluated as expected (it appears to be evaluated as **"ErrorLogModule"**
> even though the context object is **ErrorMailModule**).

Both James and Atif were very responsive in responding to these issues --
and, to be honest, that's why I waited a couple of days before writing this
post (to give them a chance to respond).

On Sunday I decided I could "live with" the issue with the JavaScript filter
in Full trust (since the Production environment for TechnologyToolbox.com
*must* run in Medium trust due to the hosting environment).

Regardless of my particular filter configuration, I'm glad to see the workarounds
that James and Atif suggested in their comments on the issues. It shows they
genuinely care about the ELMAH community -- and those of us who consider ourselves
members of this undoubtedly very large group should be very grateful for their
efforts.

I've said it before, and I certainly see no harm in saying it again: "Software
is never perfect."

ELMAH may have a few issues in a small number of scenarios, but overall,
I have to say that I consider it to be fantastic. In many ways, I even like
it better than the equivalent error handling functionality in SharePoint :-)

