---
title: Managing Email Effectively
date: 2010-01-04T07:10:00-07:00
description:
  First of all, Happy New Year! This morning I'm back from a not-so-relaxing
  four weeks off -- although I have to admit, there's something quite nice about
  putting technology aside for a few weeks and laying travertine and building
  cabinets instead ...
aliases:
  [
    "/blog/jjameson/archive/2010/01/03/managing-email-effectively.aspx",
    "/blog/jjameson/archive/2010/01/04/managing-email-effectively.aspx",
  ]
categories: ["My System"]
tags: ["My System", "Simplify"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/01/04/managing-email-effectively.aspx"
---

First of all, Happy New Year!

This morning I'm back from a not-so-relaxing four weeks off -- although I have
to admit, there's something quite nice about putting technology aside for a few
weeks and laying travertine and building cabinets instead (I am remodeling our
master bathroom). [I've done several tile projects in the past, but working with
natural stone is definitely a very different experience. It makes using ceramic
and porcelain seem somewhat trivial by comparison.]

Since the "gears" are a little rusty this morning, I thought I would start the
new year off with a blog post that, while not very deep from a technical
perspective, should nevertheless prove helpful to people like me.

Like many of you out there, I receive a lot of email on a daily basis. Now
imagine how quickly the messages pile up when returning from a nice long
vacation! I'm sure many of you can relate to the deluge of email in your inbox.

As a consultant, I find the key to effectively managing my inbox is separating
the important messages from the lower priority items. For example, email
specific to the particular project I am working on at any given time is
definitely at the top of the "important" list, whereas the numerous messages
that I receive simply by being a member of a DL (distribution list) are lower
priority.

Most of you are probably already using the **E-mail Rules** feature in Outlook
to help manage your inboxes. If not, then open Outlook, and on the **Tools**
menu, click **Rules and Alerts**.

This post details the rules that I use for ensuring my Outlook **Inbox** folder
only contains the really good stuff (and how everything else gets shuffled off
to a different location).

It is also important to note that my **Inbox** folder is synchronized to my
Windows Smartphone -- and obviously I don't want my phone "chirping" each and
every time a receive a message is sent to one of the DLs that I belong to.

For reference purposes, here's what my Outlook folder looked like first thing
this morning (again, note that this is not a typical snapshot since I just
returned from a long vacation).

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Personal/Outlook-folders-and-archive-structure-297x600.png"
alt="Outlook folders and archive structure" class="screenshot" height="600"
width="297" title="Figure 1: Outlook folders and archive structure" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Personal/Outlook-folders-and-archive-structure-334x675.png)

As you can see, I received literally thousands of email messages during my
hiatus, but only 66 of them stayed in my **Inbox** folder.

Before explaining the various Outlook rules that I have configured, let's first
cover the fundamentals of the various folders and archives illustrated above.

I have created folders under **Inbox** corresponding to various distribution
lists. I have also created an "archive" -- which is really just a .pst file
(Outlook Personal Folders) -- for any DL where I want to preserve some history.
Periodically, I move the contents of the DL folders under **Inbox** into the
corresponding archive files. [Note that I originally used a single archive file,
but found that over time these folders could grow to be very large and
subsequently split the archive into multiple files.]

I configure rules within Outlook to "preprocess" messages sent either directly
to me or to DLs that I belong to.

{{< div-block "note important" >}}

> **Important**
>
> The reason for using DL folders under the **Inbox** folder -- instead of
> having the Outlook rules route directly to the archive (.pst) files -- is to
> ensure the rules are processed on the Exchange server.
> If you configure an Outlook rule to move messages to a .pst file, then
> obviously that rule can only be processed on the client (where the .pst is
> available). While that alternate approach might work for some people, anyone
> with a smartphone configured to receive email will quickly find it to be
> rather useless.

{{< /div-block >}}

I just inspected my email rules in Outlook and found that I am now up to 32
rules. Note that most of these rules follow the same basic structure:

{{< div-block "fst-italic" >}}

> Apply this rule after the message arrives\
> sent to [people or distribution list](#)\
> move it to the [specified](#) folder\
> except if my name is in the To or Cc box\
> stop processing more rules

{{< /div-block >}}

For example, I have a rule named **SharePoint 2010 discussions** configured as
follows:

{{< div-block "fst-italic" >}}

> Apply this rule after the message arrives\
> sent to [SharePoint 2010 Discussion](#)\
> move it to the [SharePoint 2010](#) folder\
> except if my name is in the To or Cc box\
> stop processing more rules

{{< /div-block >}}

As you can see from the previous screenshot, this rule automatically routed
1,837 messages sent to the **SharePoint 2010 Discussion** DL to the
corresponding **SharePoint 2010** folder.

I include the exception where my name is in the To or Cc box in order to ensure
that any responses to messages that I send to a DL remain in the **Inbox**
folder (since I am likely looking for help or providing assistance to someone
else and therefore want to view the responses as soon as possible).

Of the 32 rules that I currently have configured, most are for DL processing.
However, there are a couple of interesting variants.

If you use Visual Studio Team Foundation Server (TFS) then you probably know
that you can subscribe to alerts -- such as when somebody checks in a changeset
for a project. I find these alerts to be helpful at times (to quickly scan and
review team progress) but obviously I don't want them clogging my inbox (or
making my smartphone "chirp" incessantly). Consequently I configured my **TFS
Notifications** rule as follows:

{{< div-block "fst-italic" >}}

> Apply this rule after the message arrives\
> from [xxx@microsoft.com](mailto:xxx@microsoft.com)\
> move it to the [TFS Notifications](#) folder

{{< /div-block >}}

Note that [xxx@microsoft.com](mailto:xxx@microsoft.com) is the email address of
the TFS service account (which I've replaced for obvious reasons).

Further down the list of email rules, I have one named **High Priority Items**:

{{< div-block "fst-italic" >}}

> Apply this rule after the message arrives\
> from
> [Ron Stutz or Scott Krebs or Sid Hayutin or Kit Ambrose or John MacCatherine](#)\
> stop processing more rules

{{< /div-block >}}

While somewhat serving as a list of various managers I've had during my tenure
with Microsoft, this rule is really used to ensure that email from these
individuals doesn't mistakenly "slip through the cracks" and get processed by
the very last rule.

Note that I also have a **Project Mail** rule configured for similar purposes:

{{< div-block "fst-italic" >}}

> Apply this rule after the message arrives\
> sent to [FrontierV3Dev or KPMG-COM Project Team or ...](#)\
> stop processing more rules

{{< /div-block >}}

The very last rule that I have configured in Outlook is named **Low Priority
Items** and is configured as follows:

{{< div-block "fst-italic" >}}

> Apply this rule after the message arrives\
> move it to the [Low Priority Items](#) folder\
> except if my name is in the To or Cc box

{{< /div-block >}}

In other words, any email message that somehow makes its way into my inbox, but
is neither processed by a previous email rule nor sent directly to me, is
summarily routed to my **Low Priority Items** folder. While I strive to catch up
on this folder every couple of days, in all honesty, it is sometimes a week or
more before I read these messages.

While I'd like to take all of the credit for this system of managing email, I
must confess that the concept of the "Low Priority" folder is one I picked up
from someone else a long, long time ago. I wish I could remember whose Webcast
that was, but it's been so long that all I remember is that it was someone from
Microsoft. Heck, for all I know, he could have pilfered the concept from
somebody else.

Lastly, I should also point out that another key element of my system for
managing email is using
[Thread Compressor](http://blogs.technet.com/ewan/archive/2007/04/23/thread-compressor-for-outlook-do-you-want-it.aspx).
This add-in is very useful for greatly reducing the number of email messages you
need to wade through (especially when returning from vacation).

Here's my updated screenshot from Outlook this morning after running Thread
Compressor:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Personal/Outlook-folders-after-Thread-Compressor-297x600.png"
alt="Outlook folders after running Thread Compressor" class="screenshot"
height="600" width="297"
title="Figure 2: Outlook folders after running Thread Compressor" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Personal/Outlook-folders-after-Thread-Compressor-334x675.png)

Compare the numbers in this screenshot with the corresponding numbers in the
previous screenshot -- or just trust me when I say it deleted 1,758 messages --
and you'll quickly see why I consider Thread Compressor to be an essential part
of my toolbox.

I hope you find these tips useful for managing your ever-increasing amount of
email.
