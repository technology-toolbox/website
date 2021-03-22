---
title: "Introducing the \"DR.DADA\" Approach to SharePoint Development"
date: 2009-03-31T09:16:00-06:00
excerpt:
  "At times, it seems like developing SharePoint solutions is all I've been
  doing since I joined Microsoft in 2000. While many things have certainly
  changed since the old \"Tahoe\" days, at least one thing remains relatively
  the same: my recommendation to..."
aliases:
  [
    "/blog/jjameson/archive/2009/03/30/introducing-the-dr-dada-approach-to-sharepoint-development.aspx",
    "/blog/jjameson/archive/2009/03/31/introducing-the-dr-dada-approach-to-sharepoint-development.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/03/31/introducing-the-dr-dada-approach-to-sharepoint-development.aspx"
---

At times, it seems like developing SharePoint solutions is all I've been doing
since I joined Microsoft in 2000. While many things have certainly changed since
the old "Tahoe" days, at least one thing remains relatively the same: my
recommendation to use as much out-of-the-box (OOTB) as possible.

I suppose it's more of a principle -- or design goal -- than a mere
recommendation, and it certainly applies to more than just Microsoft Office
SharePoint Server (MOSS) 2007 and Windows SharePoint Services (WSS). It seems
like ever since I started working with
[COTS](http://en.wikipedia.org/wiki/Commercial_off-the-shelf) solutions back in
[my PDM days at AT&T](/blog/jjameson/2007/03/03/who-is-this-guy),, the ideology
of minimal customization has just been something I try to apply to every project
I'm involved with. As I've mentioned before, it certainly makes those support
calls go much, much faster.

It is this mantra that eventually led to what I now refer to as the "DR.DADA"
approach to SharePoint development. [Note that I can't take any credit for the
"DR.DADA" moniker. That goes to a much wittier teammate of mine from my previous
project. When Ron Tielke joined my previous project, I introduced him to my
Deactivate/Retract/Delete/Add/Deploy/Activate scripts for deploying our
SharePoint features and he immediately coined it "DR.DADA" -- a name which has
stuck for almost two years.]

The main concept behind the DR.DADA approach to SharePoint development is that
you create "thin" features that build on the OOTB features in MOSS 2007 and WSS
v3 -- leveraging as much as you possibly can -- and use the various stsadm.exe
deployment operations to manage your features. Customization of SharePoint is
typically performed upon feature activation. After your features are activated,
the end result is often a site that could have been created and configured using
all OOTB functionality (although it certainly would have been tedious to do this
manually).

Note that back in the old days of WSS v2 and SharePoint Portal Server (SPS)
2003, we didn't have the concept of SharePoint solutions and features (at least
not as we refer to them today). Consequently, customization of WSS v2 and SPS
2003 typically involved creating custom Web Parts and (shudder) site
definitions. One of the things that still sticks with me from those days is that
I really don't like CAML. Uh...perhaps that didn't exactly come out the way that
intended it to.

Actually, I like the _idea_ of CAML and I have to admit that it is _incredibly_
powerful. I just don't want to dive into thousands of line of XML just to figure
out why something isn't working the way I need it to. In my opinion, CAML is one
of those things that is best left to the gurus on the SharePoint Product Team
that either invented it, or tend to work with it on a daily basis.

The other thing that has always really bothered me about CAML -- and thus custom
site definitions -- is the upgrade story (or rather the lack thereof). Back in
SPS 2003 -- and I believe this still holds true today in MOSS 2007-- you can't
change your site definition (e.g. the CAML that defines your various lists and
libraries) once a site has been provisioned using that site definition. Okay,
technically speaking, you might be able to change the XML and get it to work,
but I am pretty sure you are "unsupported" at that point.

This was one of the whole reasons behind the concept of features in WSS v3 and
MOSS 2007. SharePoint "features" enable you to break apart the monolithic CAML
site definitions into discrete items.

Thus, when I started using MOSS 2007 a couple of years ago, I avoided custom
site definitions like the plague and instead started creating features. In my
early days of MOSS 2007, I also looked at the Visual Studio Extensions for WSS
(VseWSS) and, while I liked the idea of easily generating list definitions and
the like through XML, I never got over the fact that it felt a lot like creating
custom site definitions in SPS 2003. There was also no "upgrade story" for
VseWSS-generated items.

For example, suppose you want to add a column to a list that has been
instantiated on several hundred sites. VseWSS can't do that. When I mentioned
this to a group of SharePoint Program Managers while reviewing my presentation
for TechReady8 (an internal training conference for the Microsoft field), one of
the guys mentioned that this is something that is supposed to be addressed in
the next version of VssWSS. However, I haven't looked into it, so I can't say
for sure.

Abandoning custom site definitions and VseWSS, I instead guided our team down
the path of doing all of our customization through thin features and, yep, you
guessed it...code.

You say your Document Cart feature needs a list to store the items in each
individual's cart? No problem, just modify the
[FeatureConfigurator](/blog/jjameson/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator)
for the Fabrikam.Project1.DocumentCart feature to programmatically create and
configure the list upon feature activation.

What's that? We've deployed the DocumentCart feature to Production and now we
need to add a column to the list that contains the document cart items? No
problem...just modify the FeatureConfigurator to add a column to the list upon
feature activation. Thus when we "DR.D" (deactivate/retact/delete) the old
version of the DocumentCart feature and "ADA" (add/deploy/activate) the new
version of the DocumentCart feature, the list will be updated accordingly -- in
each and every environment. It's almost like "magic" -- if we ignore the fact
that some developer had to add a tiny bit of code to the feature:

```C#
      SharePointHelper.AddOrUpdateField(list, "File Icon", "FileIcon",
         SPFieldType.Text, false);
```

Note that stsadm.exe also has the **upgradesolution** operation -- which is
certainly useful in some situations; in particular, I believe it helps resolve
issues with unghosted files in certain situations (but that should be the
subject of another post altogether). However, note that the kinds of "upgrades"
you can do with the **upgradesolution** operation are severely limited. For
example, you cannot add new files to a feature and use the **upgradesolution**
operation. For more details on the limitations of the **upgradesolution**
operation, refer to the
[Upgrading a Solution](http://msdn.microsoft.com/en-us/library/aa543659.aspx)
article on MSDN.

Redeploying your features may seem like overkill -- even using scripts like
**Redeploy Features.cmd**, which I'll share in a follow-up post -- and the truth
is sometimes it is. However, keep in mind that if you -- as the developer --
know that you haven't changed anything but code since your last build and
deployment, then you can just GAC the updated assemblies, recycle your
application pool, and you are off to the races -- or at least you will be once
your SharePoint site has warmed up again!

{{< div-block "note update" >}}

> **Note**
>
> Refer to the following blog post for more details on the "DR.DADA" approach to
> SharePoint development:
>
> {{< reference title="Sample Walkthrough of the DR.DADA Approach to SharePoint" linkHref="/blog/jjameson/2009/09/28/sample-walkthrough-of-the-dr-dada-approach-to-sharepoint" linkText="http://blogs.msdn.com/b/jjameson/archive/2009/09/28/sample-walkthrough-of-the-dr-dada-approach-to-sharepoint.aspx" >}}

{{< /div-block >}}
