---
title: "Creating a simple \"Contact\" form in ASP.NET (a.k.a. Building TechnologyToolbox.com, part 20)"
date: 2012-02-01T13:03:38-07:00
excerpt: "In this post, I explain the iterative approach used to create the \"Contact\" form for the Technology Toolbox website. Through a sequence of 9 discrete steps, I describe my typical development process from initial concept to what I call \"feature complete.\""
aliases: ["/blog/jjameson/archive/2012/02/01/building-technologytoolbox-com-part-20.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["Web Development"]
---

In this post, I explain the iterative approach used to create the [Contact](/Contact/default.aspx) form for the Technology Toolbox website. Through a sequence of 9 discrete steps, I describe my typical development process from initial concept to what I call "feature complete."

### Step 1: Prototype

In [part 3](/blog/jjameson/2011/10/27/building-technologytoolbox-com-part-3) of this series, I described my recommendation to create a static HTML prototype when doing virtually any kind of Web development (ASP.NET, SharePoint, etc.). While this slightly delays the start of writing the "real" code, in my experience I have found this to save considerable time overall. Using an HTML prototype, you can very quickly mock up various design alternatives, rapidly iterate on the overall look-and-feel, and develop/refine the user experience (for example, demonstrate features using client-side scripting such as jQuery).

Here is the HTML I created for the Contact form:

```
  <p>
    Fill out the contact form below and you will receive a response within one
    business day. You may also send email directly to
    <a href="mailto:info@technologytoolbox.com">info@technologytoolbox.com</a>.</p>
  <fieldset>
    <div>
      <label for="contact-fullName">
        Name</label>
      <input id="contact-fullName" maxlength="32" size="20" type="text"
        class="required" />
    </div>
    <div>
      <label for="contact-companyName">
        Company</label>
      <input id="contact-companyName" maxlength="32" size="20" type="text"
        class="required" />
    </div>
    <div>
      <label for="contact-emailAddress">
        Email Address</label>
      <input id="contact-emailAddress" maxlength="254" size="20" type="text" />
    </div>
    <div>
      <label for="contact-telephone">
        Telephone Number</label>
      <input id="contact-telephone" maxlength="32" size="20" type="text" />
    </div>
    <div>
      <label for="contact-subject">
        Subject</label>
      <input id="contact-subject" maxlength="64" size="20" type="text"
        class="required" />
    </div>
    <div>
      <label for="contact-message">
        Message</label>
      <textarea id="contact-message" rows="5" cols="60" class="required">
      </textarea>
    </div>
  </fieldset>
  <div class="button-panel">
    <button type="submit">
      Submit</button>
  </div>
```

Notice that in addition to specifying `id` attributes on the various input elements (in order to associate the corresponding labels), I also specified `class="required"` on the mandatory fields, as well as `maxlength` attributes on the various text fields. This little bit of additional work in creating the markup goes a long way in terms of prototyping the user experience. In the next two sections, I'll show you how this also helps expedite the process of converting from static HTML to ASP.NET controls and adding field validators.

> **Note**
>
> Although not shown in the sample HTML above, there are times when it might be helpful to prototype "alternate flow" scenarios as well -- for example, to demonstrate the user experience when a required field is not specified.

### Step 2: From static HTML to ASP.NET

With the HTML prototype complete (and presumably approved by Product Management at this point), the next step was to "make it real" by converting the various HTML elements into their corresponding ASP.NET controls. For example, `<label>` elements became `<asp:Label>` elements, `<input type="text">` elements became `<asp:TextBox>` elements, etc.

While there may very well be tools out there that do this conversion for you, I've never bothered to look for them. Doing this conversion manually doesn't take very long and it is a good opportunity to "double-check my work."

Be sure to convert attributes as well (such as changing `class` to `CssClass`). Once the conversion is complete, I typically do a quick {{< kbd "F5" >}} to verify the form renders as expected.

For the sake of brevity, I won't include the ASP.NET markup at this point since I'm sure you can imagine what it would look like.

### Step 3: Add field validators

The next step is to add field validators. For example, I added an `<asp:RequiredFieldValidator>` element for each control with `CssClass="required"`.

This is how I typically markup a field validator:

```
<asp:RequiredFieldValidator runat="server" ControlToValidate="contactName"
  Display="Dynamic" ErrorMessage="Name must be specified."
  Text="(required)" CssClass="validator required" ForeColor="" />
```

Specifying the `CssClass` attribute (and overriding the `ForeColor` attribute), allows the styling to be controlled via CSS. For example:

```
.validator {
  background: url('Images/icon-error-16x16.png') no-repeat;
  color: #bd1c1c;
  min-height: 16px;
  padding-left: 21px;
}
```

In addition to required fields, you may need to add other types of validators (for example, to ensure a valid email address is specified). Be sure to add a **[ValidationSummary](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.validationsummary.aspx)** control as well.

At this point, the markup for the Contact form looked like this:

```
  <p>
    Fill out the contact form below and you will receive a response within one business
    day. You may also send email directly to <a href="mailto:info@technologytoolbox.com">
      info@technologytoolbox.com</a>.</p>
  <fieldset>
    <div>
      <asp:Label runat="server" AssociatedControlID="contactName" Text="Name" />
      <asp:TextBox runat="server" ID="contactName" CssClass="required"
        MaxLength="32" size="20" />
      <asp:RequiredFieldValidator runat="server" ControlToValidate="contactName"
        Display="Dynamic" ErrorMessage="Name must be specified."
        Text="(required)" CssClass="validator required" ForeColor="" />
    </div>
    <div>
      <asp:Label runat="server" AssociatedControlID="companyName" Text="Company" />
      <asp:TextBox runat="server" ID="companyName" CssClass="required"
        MaxLength="32" size="20" />
      <asp:RequiredFieldValidator runat="server" ControlToValidate="companyName"
        Display="Dynamic" ErrorMessage="Company must be specified."
        Text="(required)" CssClass="validator required" ForeColor="" />
    </div>
    <div>
      <asp:Label runat="server" AssociatedControlID="contactEmail"
        Text="Email Address" />
      <asp:TextBox runat="server" ID="contactEmail" MaxLength="254" size="20" />
      <asp:RegularExpressionValidator runat="server"
        ControlToValidate="contactEmail" ValidationExpression="^.*?@.+\..+$"
        Display="dynamic"
        ErrorMessage="Email Address is not required, but if specified it must be valid."
        Text="(invalid)" CssClass="validator" ForeColor="" />
      <div class="field-info">
        Optional, but recommended (unless you wish to only be contacted by
        telephone).</div>
    </div>
    <div>
      <asp:Label runat="server" AssociatedControlID="contactTelephone"
        Text="Telephone" />
      <asp:TextBox runat="server" ID="contactTelephone" MaxLength="32"
        size="20" />
    </div>
    <div>
      <asp:Label runat="server" AssociatedControlID="preferredContactMethod"
        Text="Preferred Contact Method" />
      <asp:RadioButtonList runat="server" ID="preferredContactMethod"
        CssClass="radio-list" RepeatDirection="Horizontal">
        <asp:ListItem Text="Email" Selected="True" />
        <asp:ListItem Text="Telephone" />
      </asp:RadioButtonList>
    </div>
    <div>
      <asp:Label runat="server" AssociatedControlID="subject" Text="Subject" />
      <asp:TextBox runat="server" ID="subject" CssClass="required"
        MaxLength="64" size="20" />
      <asp:RequiredFieldValidator runat="server" ControlToValidate="subject"
        Display="Dynamic" ErrorMessage="Subject must be specified."
        Text="(required)" CssClass="validator required" ForeColor="" />
    </div>
    <div>
      <asp:Label runat="server" AssociatedControlID="message" Text="Message" />
      <asp:TextBox runat="server" ID="message" CssClass="required"
        TextMode="MultiLine" Rows="10" Columns="60" />
      <asp:RequiredFieldValidator runat="server" ControlToValidate="message"
        Display="Dynamic" ErrorMessage="Message must be specified."
        Text="(required)" CssClass="validator required" ForeColor="" /></div>
  </fieldset>
  <div class="button-panel">
    <asp:Button runat="server" Text="Submit" />
  </div>
  <asp:ValidationSummary runat="server" CssClass="validation-summary"
    ForeColor=""
    HeaderText="There is a problem with your request. Please correct and try again." />
```

### Step 4: Display confirmation message

At this point the form is complete from a visual perspective, but it doesn't actually do anything when you click the **Submit** button.

Rather than immediately getting into the details of storing or transmitting the contact information, I started out by looking at the overall user experience.

Typically, when you submit a form on a website, you expect some kind of acknowledgement. In this particular case, the Technology Toolbox website simply hides the form and displays a few words of gratitude in its place.

To implement this functionality, I simply wrapped the markup shown above inside an ASP.NET **Panel** control (i.e. a `<div>` element) and added a second panel with the confirmation message:

```
  <asp:Panel runat="server" ID="contactForm">
    <p>
      Fill out the contact form below ...</p>
    <fieldset>
      ...
    </fieldset>
    ...
  </asp:Panel>
  <asp:Panel runat="server" ID="confirmationPanel" Visible="false">
    <p>
      Thank you.</p>
    <p>
      Your contact request has been submitted. You will receive a response
      within one business day.</p>
  </asp:Panel>
```

Then it was simply a matter of associating a little bit of code with the "click" event of the **Submit** button to hide the first panel and show the second panel:

```
        protected void SubmitButton_Click(
            object sender,
            EventArgs e)
        {
            if (this.IsValid == false)
            {
                return;
            }

            // TODO: Send contact request email

            contactForm.Visible = false;
            confirmationPanel.Visible = true;
        }
```

Some quick {{< kbd "F5" >}} testing to ensure everything worked as expected and I was ready for the next step.

### Step 5: Send email message

Depending on the size of your organization, you may be able to simply email the contact request to one or more individuals and be done with it. This is the way the feature currently works on TechnologyToolbox.com.

> **Note**
>
> Sending the contact request via SMTP isn't as "bulletproof" as, say, writing the contact request to a database in real-time -- but it is probably "good enough" for many organizations. Obviously if your site is fielding dozens or hundreds of contact requests per day, then you should probably consider something a little more robust (such as saving the contact request to a database and sending some kind of notification to the appropriate people).
>
> The [Agilent LSCA](http://www.chem.agilent.com) site, for example, uses SharePoint and InfoPath Forms Services for the "[Contact Us](http://www.chem.agilent.com/en-US/ContactUs/_layouts/agilent/contactusquery.aspx?XsnLocation=/FormServerTemplates/ContactUsQueryRequest.xsn&Source=/en-US/ContactUs/Pages/ContactUs.aspx&DefaultItemOpen=1&m=p)" feature -- complete with "routing" rules on the backend to send the request to the appropriate people.

To implement the basic functionality for sending the email from TechnologyToolbox.com, I replaced the "TODO" comment shown above with a call to a new method that sends the email:

```
        private void SendContactRequestEmail()
        {
            string body = BuildEmailMessageBody();

            string mailFrom = "no-reply@technologytoolbox.com";
            string mailTo = "info@technologytoolbox.com";

            using (MailMessage mailMessage = new MailMessage(
                mailFrom,
                mailTo,
                "Contact Request - " + subject.Text,
                body))
            {
                SmtpClient client = new SmtpClient();
                client.UseDefaultCredentials = true;
                client.Send(mailMessage);
            }
        }
```

Thanks to built-in functionality in the .NET Framework, it doesn't require much code at all. Note that what you see above is the end result at this point after a little refactoring. Here is the **BuildEmailMessageBody** method:

```
        private string BuildEmailMessageBody()
        {
            StringBuilder buffer = new StringBuilder();

            buffer.AppendFormat(
                CultureInfo.CurrentCulture,
                "Name: {0}" + Environment.NewLine,
                contactName.Text);

            buffer.AppendFormat(
                CultureInfo.CurrentCulture,
                "Company: {0}" + Environment.NewLine,
                companyName.Text);

            buffer.AppendFormat(
                CultureInfo.CurrentCulture,
                "Email Address: {0}" + Environment.NewLine,
                contactEmail.Text);

            buffer.AppendFormat(
                CultureInfo.CurrentCulture,
                "Telephone: {0}" + Environment.NewLine,
                contactTelephone.Text);

            buffer.AppendFormat(
                CultureInfo.CurrentCulture,
                "Preferred Contact Method: {0}" + Environment.NewLine,
                preferredContactMethod.Text);

            buffer.Append(Environment.NewLine);

            buffer.Append(message.Text);

            return buffer.ToString();
        }
```

Testing the solution at this point revealed some issues. Specifically, sending an email from the development environment (internal) to the SMTP server provided by WinHost required some additional configuration.

The first problem was that the ISP provider I use for my home office blocks outbound SMTP by default (to prevent spam). However, by specifying a different port in Web.config, email messages can be successfully sent from Development and Test environments to the Production SMTP server.

The second problem I discovered was the email address I originally specified (info@technologytoolbox.com) is configured for [greylisting](http://en.wikipedia.org/wiki/Greylisting). After creating an email alias (with greylisting disabled) -- and modifying the code to send to that alias instead -- I was able to verify that email messages from the Contact form were received as expected.

### Step 6: Make the email settings configurable

Hard-coded values (such as the email addresses shown above) are typically a bad thing. Therefore the next step was to refactor the email addresses into corresponding settings in Web.config.

Personally, I prefer to use the [application settings](http://msdn.microsoft.com/en-us/library/cftf714c.aspx) introduced in Visual Studio 2005 (rather than the **appSettings** element included in the original version of the .NET Framework). Consequently, I created a couple of items under the **Settings** tab on the project properties:

- ContactFormMailFromAddress
- ContactFormMailToAddress

...and updated the code accordingly:

```
        private void SendContactRequestEmail()
        {
            string body = BuildEmailMessageBody();

            string mailFrom = Settings.Default.ContactFormMailFromAddress;
            string[] mailTo = Settings.Default.ContactFormMailToAddress;

            ...
        }
```

With those changes, the email addresses can be updated as needed in the Web.config file.

At this point, I started thinking about ways to "break" the solution. For example, what if instead of specifying a single "To" email address, we need to specify multiple recipients (and, for whatever reason, we don't want to use a distribution list)? To support that scenario, the **ContactFormMailToAddress** would contain a semicolon-delimited list of email addresses. For example:

```
<configuration>
  ...
  <applicationSettings>
    <TechnologyToolbox.Caelum.Website.Properties.Settings>
      ...
      <setting name="ContactFormMailToAddress" serializeAs="String">
        <value>info@technologytoolbox.com;foo@technologytoolbox.com</value>
      </setting>
    </TechnologyToolbox.Caelum.Website.Properties.Settings>
  </applicationSettings>
</configuration>
```

Note that this requires a few tweaks to the code used to send the email:

```
    private void SendContactRequestEmail()
    {
        string body = BuildEmailMessageBody();

        string mailFrom = Settings.Default.ContactFormMailFromAddress;
        string[] mailTo = Settings.Default.ContactFormMailToAddress.Split(
            new char[] { ';' });

        using (MailMessage mailMessage = new MailMessage(
            mailFrom,
            mailTo[0],
            "Contact Request - " + subject.Text,
            body))
        {
            for (int i = 1; i < mailTo.Length; i++)
            {
                if (string.IsNullOrEmpty(mailTo[i]) == false)
                {
                    mailMessage.To.Add(mailTo[i]);
                }
            }

            SmtpClient client = new SmtpClient();
            client.UseDefaultCredentials = true;
            client.Send(mailMessage);
        }
    }
```

Another quick round of testing at this point verified the enhancements function as expected.

### Step 7: Add CAPTCHA control to help prevent spam

In a [previous post](/blog/jjameson/2012/01/25/building-technologytoolbox-com-part-16), I described the custom CAPTCHA control developed for the Technology Toolbox site to support both the Contact form as well as the form used to add comments on blog posts managed by Subtext.

In hindsight, I suppose I should have written this post first, followed by the other post detailing the CAPTCHA control. However, there's a reason this blog is called the "*Random* Musings of Jeremy Jameson" ;-)

No point in repeating what I've already covered before. If you want to know the details of the CAPTCHA control, take a break and go read that post. Then come back when you are ready.

### Step 8: Enhanced field validators

At this point, the feature was nearly complete. Prospective clients could submit a request using the Contact form, which subsequently sends an email to one or more recipients specified in Web.config. The form also prevents spam from being submitted via an image-recognition CAPTCHA that is easy-to-use for people, yet sufficiently difficult to prevent spamming bots. [In the time since TechnologyToolbox.com went live nearly five months ago, I've received about a dozen emails generated via the Contact form, but none of them have been from bots.]

However, at this point, there was still a minor issue with the form regarding the **Preferred Contact Method** and corresponding **Email Address** and **Telephone** fields. Up to this point, I had not made either of these fields mandatory, based on the reasoning that some people may want to specify *only* their email addresses, while others may want to specify *only* their telephone numbers.

Yet, if you tell me that your "preferred contact method" is **Telephone**, but mistakenly forget to provide a telephone number, then that's really not a good thing. Likewise, if you say you prefer an email response but don't specify an email address, then I really can't honor your wishes.

To avoid these scenarios, I added a couple more **RequiredFieldValidator** controls to the form (one for the **Email Address** field and the other for the **Telephone** field). I also changed the default option for **Preferred Contact Method** to **Telephone**. [I don't know about you, but I'd much rather start a business relationship with a live conversation rather than exchanging emails. Email is a wonderful tool, but it also tends to slow things down a little and -- on rare occasions -- can result in misunderstandings (or too much "reading between the lines").]

The key to adding these validators is enabling/disabling them so that, for example, a telephone number does not need to be specified when people indicate they wish to be contacted via email.

To accomplish this, I created a simple method to configure the form fields based on the currently selected option for **Preferred Contact Method**:

```
        private void ConfigureFormFields()
        {
            switch (preferredContactMethod.Text)
            {
                case "Email":
                    contactEmail.CssClass = "required";
                    contactTelephone.CssClass = null;

                    contactEmailRequiredValidator.Enabled = true;
                    contactTelephoneRequiredValidator.Enabled = false;
                    break;

                case "Telephone":
                    contactEmail.CssClass = null;
                    contactTelephone.CssClass = "required";

                    contactEmailRequiredValidator.Enabled = false;
                    contactTelephoneRequiredValidator.Enabled = true;
                    break;

                default:
                    throw new InvalidOperationException(
                        "Unexpected option specified for preferred contact"
                            + " method.");
            }
        }
```

To ensure the fields are initially configured as expected, I call the **ConfigureFormFields** method when the page is first requested:

```
        protected void Page_Load(
            object sender,
            EventArgs e)
        {
            if (this.IsPostBack == false)
            {
                ConfigureFormFields();
            }
        }
```

Then I modified the **RadioButtonList** to automatically postback when the selected item is changed:

```
  <asp:RadioButtonList runat="server" ID="preferredContactMethod"
    CssClass="radio-list" RepeatDirection="Horizontal"
    AutoPostBack="true"
    OnSelectedIndexChanged="PreferredContactMethod_SelectedIndexChanged">
    <asp:ListItem Text="Email" />
    <asp:ListItem Text="Telephone" Selected="True" />
  </asp:RadioButtonList>
```

...and subsequently call the **ConfigureFormFields** method:

```
        protected void PreferredContactMethod_SelectedIndexChanged(
            object sender,
            EventArgs e)
        {
            ConfigureFormFields();
        }
```

Another quick round of testing to verify the enhancements functioned as expected, followed by another check-in of the updated code.

### Step 9: Use AJAX to improve the user experience

Okay, *almost* done...

While all of the functionality was complete at this point, there was still a little room for improvement. Rather than doing a full postback whenever the **Preferred Contact Method** is changed, I added an **UpdatePanel** to perform a partial page update instead. In other words, I sprinkled a little AJAX onto the Contact form once it was almost completely "baked."

Generally speaking, this is how I approach AJAX-enabling a feature:

1. Get the feature working as expected (without any AJAX)
2. Refine the solution to improve the user experience (by adding AJAX)
3. Optimize for performance

The third item is important. For example, for step "9.2" you might start by adding an **UpdatePanel** around all of the form fields:

```
        <asp:Panel runat="server" ID="contactForm">
          <asp:UpdatePanel runat="server">
            <ContentTemplate>
              <p>
                Fill out the contact form below...</p>
              <fieldset>
                ...
              </fieldset>
              ...
            </ContentTemplate>
          </asp:UpdatePanel>
        </asp:Panel>
```

However, in this particular scenario, only the **Email Address** and **Telephone** fields (and corresponding validators) are updated by the AJAX postback. Consequently, it is much more efficient to enclose only those controls in the **UpdatePanel**:

```
        <asp:Panel runat="server" ID="contactForm">
          <p>
            Fill out the contact form below...</p>
          <fieldset>
            ...
            <div>
              <asp:Label ... Text="Company" />
              <asp:TextBox ... ID="companyName" />
              <asp:RequiredFieldValidator ... ControlToValidate="companyName" />
            </div>
            <asp:UpdatePanel runat="server">
              <ContentTemplate>
                <div>
                  <asp:Label ... Text="Email Address" />
                  <asp:TextBox ... ID="contactEmail" />
                  <asp:RegularExpressionValidator ControlToValidate="contactEmail" ... />
                  <asp:RequiredFieldValidator ControlToValidate="contactEmail" ... />
                  ...
                </div>
                ...
              </ContentTemplate>
            </asp:UpdatePanel>
            <div>
              <asp:Label ... Text="Subject" />
              <asp:TextBox ... ID="subject" />
              <asp:RequiredFieldValidator ControlToValidate="subject" ... />
            </div>
            ...
          </fieldset>
          ...
        </asp:Panel>
        ...
```

You can see this for yourself by inspecting the AJAX postback using the **Network** tab in the Internet Explorer Developer Tools or via the **Net** panel in Firebug/Firefox.

