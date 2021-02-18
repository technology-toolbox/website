---
title: "Operations Manager Alerts for Event Log Errors"
date: 2011-03-18T00:14:00-07:00
excerpt: "One of the things I like most about running System Center Operations Manager in the \"Jameson Datacenter\" (a.k.a. my home lab) is that it greatly reduces the amount of effort required to monitor numerous servers. 
 For example, in my environment I am..."
aliases: ["/blog/jjameson/archive/2011/03/18/operations-manager-alerts-for-event-log-errors.aspx"]
draft: true
categories: ["My System", "Infrastructure"]
tags: ["My System", "Simplify", "Infrastructure"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/03/18/operations-manager-alerts-for-event-log-errors.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/03/18/operations-manager-alerts-for-event-log-errors.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

One of the things I like most about running System Center Operations Manager in the ["Jameson Datacenter"](/blog/jjameson/2009/09/14/the-jameson-datacenter) (a.k.a. my home lab) is that it greatly reduces the amount of effort required to monitor numerous servers.

For example, in my environment I am currently monitoring 10 servers 24x7. To some infrastructure folks out there, this might seem like a laughable number -- but keep in mind that my primary role is development, not infrastructure.

While it's nice to have a number of different physical and virtual machines available to help me in my day-to-day work, I can't spend significant time managing the environment -- or else I simply wouldn't have time to do the things that really matter (like ensuring I always deliver the features committed for a particular sprint, or spending time with my wife and daughter).

Consequently, one of the few changes I've made to the out-of-the-box SCOM 2007 R2 configuration is to create a couple of rules so I get an email whenever there is a "hiccup" on any of the monitored servers in my environment. [This is in addition to installing a number of management packs for things like Active Directory, SQL Server, SharePoint, and TFS.]

In other words, I created two rules that generate alerts whenever an error occurs in the Application event log or the System event log on *any* server being monitored by Operation Manager. While there may be errors generated in other event logs (such as the custom "Operations Manager" log that gets created when you install the SCOM agent on a server), I don't attempt to monitor every event log. My assumption is that errors in other event logs will either be detected by a suitable management pack or will manifest themselves as errors in the Application or System event log if the problem is severe enough.

When adding custom monitors and rules in Operation Manager, you first have to decide which management pack should contain the customizations. Based on what I've read, the best practice is to avoid the Default Management Pack and instead create custom management packs that are specific to a particular area. For my custom "event log error" rules, I created a new management pack called **Windows Core Library - Customizations**.

To create a rule that generates an alert whenever an error occurs in the Application event log:

1. Start the **Operations Console** as a member of the Operations Manager Authors or Administrators role.
2. In the Operations console. click the **Authoring** button.
3. In the navigation pane:
   1. Expand **Authoring**, and then expand **Management Pack Objects**.
   2. Right-click **Rules**, and then click **Create a new rule...** to start the Create Rule Wizard.
4. On the **Select a Rule Type**page:
   1. Expand **Alert Generating Rules**, expand **Event Based**, and then click **NT Event Log (Alert)**.
   2. Select the destination management from the list (**Windows Core Library - Customizations**) or click **New...** to create a management pack.
   3. Click **Next**.
5. On the **Rule Name and Description**page:
   1. In the **Rule name** box, type **Application Event Log Error**.
   2. Optionally, type a description for the rule.
   3. Click **Select** to select the item to target.
   4. In the **Select Items to Target** dialog, select **Windows Computer**, and then click **OK**.
   5. Ensure the **Rule is enabled** option is checked and then click **Next**.
6. On the **Event Log Name** page, ensure **Log name** is set to **Application**, and then click **Next**.
7. On the **Build Event Expression**page:
   1. Specify the following expression:
      
      | Parameter Name | Operator | Value |
      | --- | --- | --- |
      | Event Level | Equals | Error |
   
   2. Click **Next**.
8. On the **Configure Alerts**page:
   1. In the **Alert description** box, specify the following:
      **Source: $Data/EventSourceName$
      Event ID: $Data/EventDisplayNumber$
      Event Category: $Data/EventCategory$
      User: $Data/UserName$
      Computer: $Data/LoggingComputer$
      Event Description: $Data/EventDescription$**
   2. In the **Severity** option, click **Warning**.
   3. Click **Alert suppression...** to define the handling of duplicate alerts. In the **Alert Suppression**dialog:
      1. Click the following fields:
         - **Event ID**
         - **Event Source**
         - **Logging Computer**
         - **Event Category**
         - **User**
         - **Description**
      2. Click **OK**.
   4. Click **Create**.

Repeat the process to create a similar alert for errors in the System event log.

> **Important**
>
> If you do not specify any fields in the Alert Suppression dialog, then you may receive numerous alerts within a short period of time (for example, when SharePoint Server 2010 floods the Application event log due to an issue with least-privilege configuration).
>
> When this occurs, Operations Manager will detect the high frequency of alerts and temporarily suspend the notification, and display a different alert instead:
>
> **Alert rule:** Alert generation was temporarily suspended due to too many alerts.
>
> **Alert description:** A rule has generated 50 alerts in the last 60 seconds. Usually, when a rule generates this many alerts, it is because the rule definition is misconfigured. Please examine the rule for errors. In order to avoid excessive load, this rule will be temporarily suspended until ...

> **Note**
>
> The reason why I choose to set the **Severity** to **Warning** (instead of the default -- **Critical**) is so that when an event log error generates a similar alert in one of the other management packs, I immediately focus on the "primary" alert (rather than the "duplicate" generated by the custom rule).

In order to minimize the effort required to investigate errors in the event logs, I include details from the event in the alert. This is especially useful for quickly understanding errors on a server since it is also included in email generated by the alert.

Generating alerts for any errors that occur in the Application and System event logs will definitely motivate you to take corrective action to resolve the errors. It will also encourage you to try to prevent the same errors from occurring again in the future.

I've attached my sample management pack with the two custom rules -- just in case you want to save yourself the 5 minutes or so it takes to configure the rules.

