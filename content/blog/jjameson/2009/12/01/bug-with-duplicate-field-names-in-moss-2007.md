---
title: Bug with Duplicate Field Names in MOSS 2007
date: 2009-12-01T08:12:00-07:00
excerpt:
  I encountered a rather nasty bug in Microsoft Office SharePoint Server (MOSS)
  2007 yesterday that occurs when a custom field (i.e. site column) has the same
  name as an existing field. Note that this issue will also occur in Windows
  SharePoint Services...
aliases:
  [
    "/blog/jjameson/archive/2009/11/30/bug-with-duplicate-field-names-in-moss-2007.aspx",
    "/blog/jjameson/archive/2009/12/01/bug-with-duplicate-field-names-in-moss-2007.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/12/01/bug-with-duplicate-field-names-in-moss-2007.aspx"
---

I encountered a rather nasty bug in Microsoft Office SharePoint Server (MOSS)
2007 yesterday that occurs when a custom field (i.e. site column) has the same
name as an existing field. Note that this issue will also occur in Windows
SharePoint Services (WSS) v3.

I initially created the following Fields.xml file:

```XML
<?xml version="1.0" encoding="utf-8" ?>
<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
  <Field ID="{11F7E97D-6282-44b1-8222-2E6E377BCDFC}"
    Name="AnnouncementEndDate"
    Type="DateTime"
    Format="DateOnly"
    Group="Fabrikam Custom Columns"
    DisplayName="Announcement End Date">
  </Field>
  <Field ID="{9D1F7E52-EFBF-431e-9331-7DCA31D9CB7A}"
    Name="AnnouncementStartDate"
    Type="DateTime"
    Format="DateOnly"
    Group="Fabrikam Custom Columns"
    DisplayName="Announcement Start Date">
  </Field>
</Elements>
```

Next, I created the ContentTypes.xml file:

```XML
<?xml version="1.0" encoding="utf-8" ?>
<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
  <ContentType
    ID="0x010100C568DB52D9D0A14D9B2FDCC96666E9F2007948130EC3DB064584E219954237AF390064DEA0F50FC8C147B0B6EA0636C4A7D400EF37EB6F40C54a21A3872B1E6CA5BC0A"
    Name="Announcement Page"
    Description="Announcement Page is a custom content type ..."
    Group="Fabrikam Content Types">
    <FieldRefs>
      <FieldRef ID="{9D1F7E52-EFBF-431e-9331-7DCA31D9CB7A}" Name="AnnouncementStartDate" />
      <FieldRef ID="{11F7E97D-6282-44b1-8222-2E6E377BCDFC}" Name="AnnouncementEndDate" />
    </FieldRefs>
    <DocumentTemplate TargetName="/_layouts/CreatePage.aspx" />
  </ContentType>
</Elements>
```

{{< div-block "note" >}}

> **Note**
> 
> The really long ContentType ID above specifies that Announcement Page inherits
> from the out-of-the-box Welcome Page. I vaguely recall reading somebody's blog
> where he or she stated that you shouldn't inherit from the OOTB page types.
> However, I don't recall the justification -- if there even was one. I haven't
> encountered any problems with this approach and thus haven't seen any
> compelling reason to stop doing so.

{{< /div-block >}}

Then I created a page layout for the new Announcement Page content type.

After building and deploying the corresponding WSP -- and activating the feature
-- I verified that I could create a new Announcement Page and specify field
values, including Announcement Start Date and Announcement End Date.
Consequently I checked in my changes to TFS.

At that point, I started thinking that perhaps AnnouncementStartDate should
simply be StartDate and AnnouncementEndDate should simply be EndDate (thinking
that we might want to use these fields for more than just announcements at some
point in the future). Consequently, I changed the Fields.xml file as follows:

```XML
<?xml version="1.0" encoding="utf-8" ?>
<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
  <Field ID="{11F7E97D-6282-44b1-8222-2E6E377BCDFC}"
    Name="EndDate"
    Type="DateTime"
    Format="DateOnly"
    Group="Fabrikam Custom Columns"
    DisplayName="End Date">
  </Field>
  <Field ID="{9D1F7E52-EFBF-431e-9331-7DCA31D9CB7A}"
    Name="StartDate"
    Type="DateTime"
    Format="DateOnly"
    Group="Fabrikam Custom Columns"
    DisplayName="Start Date">
  </Field>
</Elements>
```

I propagated the changes to the ContentTypes.xml file:

```XML
<?xml version="1.0" encoding="utf-8" ?>
<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
  <ContentType
    ID="0x010100C568DB52D9D0A14D9B2FDCC96666E9F2007948130EC3DB064584E219954237AF390064DEA0F50FC8C147B0B6EA0636C4A7D400EF37EB6F40C54a21A3872B1E6CA5BC0A"
    Name="Announcement Page"
    Description="Announcement Page is a custom content type ..."
    Group="Fabrikam Content Types">
    <FieldRefs>
      <FieldRef ID="{9D1F7E52-EFBF-431e-9331-7DCA31D9CB7A}" Name="StartDate" />
      <FieldRef ID="{11F7E97D-6282-44b1-8222-2E6E377BCDFC}" Name="EndDate" />
    </FieldRefs>
    <DocumentTemplate TargetName="/_layouts/CreatePage.aspx" />
  </ContentType>
</Elements>
```

After rebuilding my Web application, adding and deploying the updated WSP, and
activating the feature, I discovered that the Announcement Page content type did
not have the Start Date and End Date fields.

Since there were no errors deploying the WSP or activating the feature, I was
rather baffled by the issue. Then I discovered that by renaming
AnnouncementStartDate to StartDate, I introduced a duplicate field name. Note
the following field definition from the OOTB fieldswss.xml file:

```XML
    <Field ID="{64cd368d-2f95-4bfc-a1f9-8d4324ecb007}"
        Name="StartDate"
        SourceID="http://schemas.microsoft.com/sharepoint/v3"
        StaticName="StartDate"
        Group="$Resources:Base_Columns"
        Type="DateTime"
        Format="DateOnly"
        DisplayName="$Resources:core,Start_Date;"><!-- _locID@DisplayName="camlidT6" _locComment=" " -->
        <Default>[today]</Default>
    </Field>
```

Similarly, my custom EndDate field conflicted with the OOTB EndDate field.

My memory might be incorrect, but I vaguely recall getting an error a few years
ago when trying to define a second field with the same name as an existing
field. Obviously that was long before MOSS 2007 Service Pack 2 (which I am
currently running) so it's possible the behavior has changed. [Honestly, while I
_could_ try to repro this using the MOSS 2007 RTM version, I'm not going to
bother. It would take me about an hour to build out a vanilla MOSS 2007 RTM
environment, and that is far longer than I am willing to invest in this issue.
It's also possible that I am simply remembering a slightly different issue --
perhaps a duplicate content type name.]

Regardless of whether this is a new problem, or has been around since the
original MOSS 2007 release isn't the point. What is important is that you get no
warnings or errors when your custom feature mistakenly attempts to define new
StartDate and EndDate fields.

Once I identified the crux of the issue, I decided to simply rip out my custom
StartDate and EndDate fields and instead use the OOTB fields. After all, as my
old mantra goes, always try to use something out-of-the-box instead of
customization whenever possible.

In other words, I updated my ContentTypes.xml file to the following:

```XML
<?xml version="1.0" encoding="utf-8" ?>
<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
  <ContentType
    ID="0x010100C568DB52D9D0A14D9B2FDCC96666E9F2007948130EC3DB064584E219954237AF390064DEA0F50FC8C147B0B6EA0636C4A7D400EF37EB6F40C54a21A3872B1E6CA5BC0A"
    Name="Announcement Page"
    Description="Announcement Page is a custom content type ..."
    Group="Fabrikam Content Types">
    <FieldRefs>
      <FieldRef ID="{64cd368d-2f95-4bfc-a1f9-8d4324ecb007}" Name="StartDate" />
      <FieldRef ID="{2684F9F2-54BE-429f-BA06-76754FC056BF}" Name="EndDate" />
    </FieldRefs>
    <DocumentTemplate TargetName="/_layouts/CreatePage.aspx" />
  </ContentType>
</Elements>
```

Unfortunately, I then discovered that the OOTB EndDate field does not parallel
the definition of the OOTB StartDate field:

```XML
    <Field ID="{2684F9F2-54BE-429f-BA06-76754FC056BF}"
        Name="EndDate"
        Type="DateTime"
        DisplayName="$Resources:core,End_Time;"
        Format="DateTime"
        FromBaseType="TRUE"
        Group="_Hidden"
        SourceID="http://schemas.microsoft.com/sharepoint/v3/fields"
        StaticName="EndDate" ><!--DisplayName=$Resources:camlid3;-->
        <FieldRefs>
            <FieldRef ID="{7D95D1F4-F5FD-4a70-90CD-B35ABC9B5BC8}" Name="fAllDayEvent" RefType="AllDayEvent" />
        </FieldRefs>
        <Default>[today]</Default>
    </Field>
```

Notice that with EndDate, `Format="DateTime"` whereas with StartDate,
`Format="DateOnly"`. Also note that the display name (in English - U.S.) is "End
Time" -- instead of the expected "End Date". So much for consistency ;-)

Lastly, note that the OOTB fieldswss.xml also contains the following:

```XML
    <Field ID="{8A121252-85A9-443d-8217-A1B57020FADF}"
        Name="_EndDate"
        Group="$Resources:Base_Columns"
        Type="DateTime"
        DisplayName="$Resources:End_Date"
        Format="DateTime"
        SourceID="http://schemas.microsoft.com/sharepoint/v3/fields"
        StaticName="_EndDate" >
        <Default>[today]</Default>
    </Field>
```

Unfortunately, this field definition doesn't specify `Format="DateOnly"` either
-- otherwise I would have just used it instead.

Thus I reverted back to using my original AnnouncementStartDate and
AnnouncementEndDate custom fields. It's not ideal, but it works.

