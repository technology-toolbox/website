---
title: "Dell, your feedback feature is a little aggravating."
date: 2022-01-23T09:49:05-07:00
categories: ["Development"]
description: Why would a "Submit Feedback" feature block parentheses?!
images:
  [
    "https://assets.technologytoolbox.com/screenshots/36/FEEE6EA2FF806EC92F72D216E76799AAD8C2F336.png",
  ]
tags: ["Web Development"]
---

This morning I
[googled _windows install date_](https://www.google.com/search?q=windows+install+date)
to determine when I last rebuilt my desktop. This was the top result:

{{< reference title="How to Use Command Prompt to Verify Install Date of Microsoft Windows" linkHref="https://www.dell.com/support/kbdoc/en-us/000131940/how-to-use-command-prompt-to-verify-install-date-of-microsoft-windows" >}}

Unfortunately, both methods presented in the Dell article for finding the
install date of Windows give incorrect information (at least in my particular
case).

> **Important:** Never assume the **Created** date for the **C:\Windows** folder
> corresponds to the time you originally installed Windows. That _may_ have
> worked at some point in the past, but today that's just plain _wrong_.
> Similarly -- as mentioned in the Dell article -- don't trust the **Original
> Install Date** in the output from `systeminfo` either. More than likely, that
> value corresponds to a major Windows 10 update (e.g. upgrading from Windows 10
> version 1909 to Windows 10 version 2009).

Since the article is the top search result on Google, I attempted to submit
feedback in hopes that Dell might update the content to address the issues.
That's when I encountered the following:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/36/FEEE6EA2FF806EC92F72D216E76799AAD8C2F336.png"
  class="screenshot" height="793" width="1266"
  caption="Figure 1: Error submitting feedback on Dell website" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/36/FEEE6EA2FF806EC92F72D216E76799AAD8C2F336.png)

As you can see, the feedback feature on the Dell website does not accept
comments with "special characters: `<>()\`" -- which blocked me from submitting
the following feedback:

```Text
Both of the methods presented here give the wrong install date when Windows 10
has been upgraded (e.g. 1909 --> 2009 --> 21H2).

The following PowerShell gives the actual install date for Windows:

@(Get-ChildItem -Path HKLM:\System\Setup\Source* |
    foreach {Get-ItemProperty -Path Registry::$_}; Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion') |
    select ProductName, ReleaseID, CurrentBuild,
        @{Name='InstallDate';
          Expression={[TimeZone]::CurrentTimeZone.ToLocalTime(
            ([DateTime]'1/1/1970').AddSeconds($_.InstallDate))}} |
    sort InstallDate

Reference:

https://www.digitalcitizen.life/simple-questions-when-was-windows-installed-my-computer/
```

Now, I can _sort of_ understand why the Dell website does not want you to enter
angle brackets -- specifically, to avoid potential issues with HTML content --
but not accepting _parentheses_?! _Really_?!

As a _user_, it's not my responsibilty to figure out how to avoid "special
characters" or "escape" content when entering data; that's a _developer's_ job.
If you can't figure out how to safely support angle brackets, parentheses, and
backslashes in content submitted via the web...well, then maybe you shouldn't
consider yourself a _qualified_ web developer.

Dell, please "enhance" your feedback system to fix this issue -- or
[hire a consultant who can easily do this for you](https://www.technologytoolbox.com/)
;-)
