---
title: Tell Mode vs. Ask Mode
date: 2009-06-03T19:11:00-06:00
excerpt:
  "The project I am currently working on is nearing the end. Last week we
  reached our \"Feature Complete\" milestone and now we have formally
  transitioned into the \"Stabilizing\" phase. In a couple of team meetings this
  week, I mentioned the concepts of..."
aliases:
  [
    "/blog/jjameson/archive/2009/06/03/tell-mode-vs-ask-mode.aspx",
  ]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/06/03/tell-mode-vs-ask-mode.aspx"
---

The project I am currently working on is nearing the end. Last week we reached
our "Feature Complete" milestone and now we have formally transitioned into the
"Stabilizing" phase.

In a couple of team meetings this week, I mentioned the concepts of "Tell Mode"
and "Ask Mode" -- a couple of terms I've been using for more years than I can
remember. However, not everyone on the team was aware of these concepts.
Consequently, I "recycled some bits" (meaning I searched through my e-mail
archive and found a message I sent awhile back) in order to provide some more
background and details on Tell Mode and Ask Mode.

I thought this worth sharing with a broader audience.

I'll warn you...this is a rather long post and not one I'm expecting many people
to read in its entirety. Rather, I think a lot of people would benefit from
reading just enough of my e-mail to understand the key concepts and perhaps skim
the example bugs to get a feel for the level of detail expected during the
"investigation" phase after switching to Ask Mode.

> * * *
> 
> **From:** JAMESON,JEREMY (Non-A-BPI-AM,unix1)\
> **Sent:** Thursday, October 18, 2007 8:03 AM\
> **Subject:** v2.0 Transition from "Tell Mode" to "Ask Mode"
> 
> On Sunday night, October 21st, RC4 of the _[Project]_ v2.0 solution will be
> deployed to TEST. This will mark the official transition from "Tell Mode" to
> "Ask Mode."
> 
> What does this mean?
> 
> Microsoft uses Tell Mode and Ask Mode to refer to different time periods after
> the Feature Complete milestone but prior to a release.
> 
> During Tell Mode the Development team _tells_ the rest of the team which
> issues are actively being worked on (resolved). In other words, Development is
> primarily driving the stabilization process based on what they think should be
> fixed -- as determined by priority, severity, complexity (i.e. risk), and
> effort required. This works for a while, but at some point you have to reign
> in Development in order to ship, er, release.
> 
> Thus we transition into Ask Mode whereby Development must _ask_ permission
> before resolving any issues. It is okay to investigate issues, but <u>no code
> changes can be made</u> to address an issue until approval by the "War Team"
> (er, Triage Team) is granted.
> 
> During Ask Mode we look first at the scenario that is being fixed (i.e. the
> motivation for fixing it), we look at why the bug happens (regression, test
> hole, coding error, etc.), and then we investigate the necessary source code
> changes and associated risk. Every potential change is heavily scrutinized to
> evaluate whether it is worth the risk to fix it, because <u>at this point in
> the schedule, every change -- even one that seems trivial -- has the potential
> to destabilize the solution and slip the schedule</u>.
> 
> While the overall process of the daily triage meetings stays the same as we
> transition from Tell Mode to Ask Mode, there is a noticeable shift in focus.
> 
> During Tell Mode, work items are often immediately triaged as **Approved**, or
> work items triaged as **Investigate** are often just fixed without further
> review. This is no longer the case once we enter Ask Mode.
> 
> In Ask Mode every work item is first assigned a triage of **Investigate**. The
> person responsible for investigating the item must thoroughly document details
> about the item -- in the **Description** field of the work item, unless a more
> formal DCR (Design Change Request) is deemed necessary.
> 
> The investigator (typically a member of the Development team) documents the:
> 
> - Motivation -- "Why are we making this change?"
> - Proposal -- "How should we resolve the issue?"
> - Risks -- "What is the worst case scenario if we make this change?"
> - Solution -- "What specifically needs to change?"
> - Teams Impacted -- "How much work is required by Development, Test, Release
>   Management, and (potentially) Product Management teams?" (note that the
>   estimates need to come from the respective team members -- the investigator
>   should not "guess")
> 
> Note that both **Solution** and **Teams Impacted** are not needed, one or the
> other is fine. If the necessary change (and associated risk) is very small --
> such as a configuration change -- then just documenting the **Solution**
> should suffice. The goal is not to overburden ourselves with process -- but we
> also need to ensure that no change is made unless it has been thoroughly
> evaluated. If the change (or risk) is substantial, then document the impact on
> each of the various teams.
> 
> You can see some example bugs from v1.0 for more details:
> [1592](https://extranet.fabrikam.com/sites/Project1/Lists/Work%20Items/DispForm.aspx?ID=1592),
> [1505](https://extranet.fabrikam.com/sites/Project1/Lists/Work%20Items/DispForm.aspx?ID=1505),
> [1533](https://extranet.fabrikam.com/sites/Project1/Lists/Work%20Items/DispForm.aspx?ID=1533),
> [1582](https://extranet.fabrikam.com/sites/Project1/Lists/Work%20Items/DispForm.aspx?ID=1582),
> and
> [1549](https://extranet.fabrikam.com/sites/Project1/Lists/Work%20Items/DispForm.aspx?ID=1549).
> 
> After documenting these details, the investigator then changes the **Triage**
> field to **Recommend Approve** (if the value in making the change is greater
> than the associated risk) or **Recommend Reject** (if the risk of making the
> change outweighs the value). The Triage Team then reviews the detailed
> information about the work item and either changes the **Triage** field to
> **Approved** or **Not Approved** (i.e. "punt to v.Next").
> 
> Note that the Triage Team can certainly "overrule" the investigator, which is
> why it is critical that no work be done on actually resolving the work item
> until approval is received from the Triage Team (i.e. the **Triage** field is
> set to **Approved**). Also note that, unless you are a member of the Triage
> Team -- and even then, only during a formal triage meeting -- <u>you should
> not change the <b>Triage</b> field to anything except <b>Recommend Approve</b>
> or <b>Recommend Reject</b></u>. Otherwise, you should fully expect a thorough
> hazing from virtually all team members.
> 
> Everyone who was involved in v1.0 knows that we didn't strictly follow this
> process in the first release of _[Project]_, but we also have the "scars" to
> remind us that we need to improve our process for this release.
> 
> It is also important to note that Microsoft fully acknowledges that we had
> many more QFEs for v1.0 than we intended, and we never did deploy v1.1 as a
> separate release. We need to be committed to ensuring that we don't repeat
> this in v2.
> 
> Yes, we will release v2.0 with known issues. By now, everyone has heard
> Jeremy's anecdote about the 64,000 "issues" in the RTM version of Windows 2000
> -- so no sense repeating that. Rather we need to focus on making sure that
> most of the "issues" are known, we understand the impact of those issues on
> the user experience, and -- where necessary -- we find creative ways to
> circumvent the issues until the underlying changes can be implemented, tested,
> and deployed.
> 
> * * *

Note that I have replaced the project code name with
_[Project]_ in order to protect the innocent. I also "spoofed" the links to the referenced bugs, since these referred to a ["Work Items" list](/blog/jjameson/2008/04/01/tfs-lite-for-wss-v2)
on a secured team site we used for the project (which obviously you wouldn't
have access to -- unless you just happen to be one of my cohorts on the
project).

I have included slightly "scrubbed" versions of a couple of the bugs below, in
case you are interested in seeing examples. If you don't understand all of the
acronyms, terms, and technical details...don't worry, you're not supposed to --
unless you actually were a member of this project ;-)

### Bug 1592 - LiteratureResults.asp is not working

> #### Motivation:
> 
> There appear to be (at least) two problems when following the repro steps:
> 
> 1. The **Literature Summary** list (rendered by LiteratureResults.asp when
>    isortorder=9) should show the number of publications matching the specified
>    criteria, grouped by **Literature Type** (a.k.a. publication type, a.k.a.
>    Content Type in SharePoint, a.k.a. the ContentType2 managed property). The
>    list should be ordered by **Literature Type**.
> 2. Clicking on a publication type in the Literature Summary list should show
>    the search results for publications of the selected type. According to the
>    original description for this bug, an error occurred at this point.
> 
> #### Proposal:
> 
> Correct the **Literature Summary** view to properly group and order by
> publication type. Increase the maximum number of search results in this
> scenario from 200 to 1000.
> 
> Ensure that clicking a publication type in the list shows the matching
> publications of the specified type.
> 
> #### Risk:
> 
> The current implementation of LiteratureResults.asp uses a hard-coded maximum
> of 200 search results (this limit was chosen for performance reasons since
> there is significant cost in sending large result sets from the SharePoint
> farm to the legacy ASP farm). Therefore, simply changing the sort order to
> publication type could potentially truncate the **Literature Summary** view
> such that it only showed one publication type (for example, if more than 200
> Applications matched the specified criteria).
> 
> While it is simple to increase the maximum number of results (for example,
> from 200 to 1000) when rendering the **Literature Summary** view, this will
> put considerable load on SharePoint Search for two reasons:
> 
> 1. We need to order first by ContentType2 in order to return the result set in
>    the order expected by the legacy ASP code that generates the **Literature
>    Summary** view.
> 2. SharePoint Search is optimized to sort by rank first (since it is typically
>    desired to show the "best" results at the top of the list). In order to
>    return a result set of, say, 1000 items sorted by something other than
>    rank, SharePoint must do a lot more work before it can trim the result set.
> 
> Therefore it is quite possible that SharePoint Search will require &gt; 10
> seconds to generate these large result sets (although it is expected that
> duplicate searches will be cached and therefore return the result set much
> faster). As noted in bug 1151, this may cause "sporadic errors" in the legacy
> ASP pages.
> 
> #### Teams Impacted
> 
> **Development**\
> Modify LiteratureSearchResults.asp to change the max number of results to be a
> variable with a default value of 200.
> 
> Modify SetSortParameters in SearchModule.inc to set the sort expression used
> in ESI (strSortExpression) and to override the default max results value to
> allow 1000 results. (1 hour).\
> \
> **Release Management**\
> Merge updated ASP files into legacy VSS and deploy to WCOSLSD and CAGCHEM (1
> hour)\
> \
> **Test**\
> Retest ESI Chromatogram searches through legacy General Site (2 hours)

### Bug 1505 - There are no results displayed in WCOSLSCD if a search is performed on few publication types

> #### Repro steps:
> 
> 1. Log on to
> [http://wcoslscd.cos.fabrikam.com/scripts/library.asp](http://wcoslscd.cos.fabrikam.com/scripts/library.asp)
> 2. Select the publication type as "Certificate of Analysis" or "Material
>    Safety Data Sheet" from the drop down and click on search
> 3. There are no results displayed for these publication types even though
>    there are lot of documents under these publication types in TEST
> 
> #### Motivation:
> 
> LiteratureResults.aspx currently excludes the following content types (since
> the number of publications of these types is disproportionately higher than
> other publication types and also because these publication types have separate
> search forms):
> 
> - Chromatogram
> - Certificate of Analysis
> - Material Safety Data Sheet
> 
> However, the current user experience is not desirable since the search form
> allows users to narrow their search results to just Certificate of Analysis or
> MSDS publications (which will never return results).
> 
> #### Proposal:
> 
> Remove Certificate of Analysis and MSDS from the search criteria. This would
> allow users to search specifically by these publication types and would also
> include these results by default.
> 
> #### Risk:
> 
> Of the roughly 33,000 publications in the EPI Warehouse, there are
> approximately 1,300 Certificate of Analysis and 7,100 MSDS publications.
> 
> Since the new SharePoint search includes full-text indexing as well as
> metadata, including these two publication types by default may dramatically
> change the search results.
> 
> #### Teams Impacted
> 
> **Development**\
> Modify LiteratureResults.asp to no longer exclude Certificate of Analysis and
> MSDS publications by default (0.5 hours).
> 
> **Release Management**\
> Merge updated ASP file into legacy VSS and deploy to WCOSLSCD and CAGCHEM (0.5
> hours)
> 
> **Test**\
> Retest ESI Library searches through legacy General Site (2 hours)
> 
> **Product Management**\
> Review new Library search results on legacy General Site to determine if the
> large number of Certificate of Analysis and MSDS publications has a negative
> impact on search results. (2 hours)
