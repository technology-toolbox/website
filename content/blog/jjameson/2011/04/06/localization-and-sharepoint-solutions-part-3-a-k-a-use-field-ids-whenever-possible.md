---
title: Localization and SharePoint Solutions, Part 3 (a.k.a. use field IDs whenever possible)
date: 2011-04-06T04:45:00-06:00
excerpt:
  In part 1 of this series , I mentioned that one of the options for creating
  SharePoint sites in multiple languages is to install the corresponding
  SharePoint language packs prior to creating the sites. This is the most common
  deployment scenario for localization...
aliases:
  [
    "/blog/jjameson/archive/2011/04/05/localization-and-sharepoint-solutions-part-3-a-k-a-use-field-ids-whenever-possible.aspx",
    "/blog/jjameson/archive/2011/04/06/localization-and-sharepoint-solutions-part-3-a-k-a-use-field-ids-whenever-possible.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "SharePoint 2010"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2011/04/06/localization-and-sharepoint-solutions-part-3-a-k-a-use-field-ids-whenever-possible.aspx"
---

In
[part 1 of this series](/blog/jjameson/2010/10/25/localization-and-sharepoint-solutions-part-1),
I mentioned that one of the options for creating SharePoint sites in multiple
languages is to install the corresponding SharePoint language packs prior to
creating the sites. This is the most common deployment scenario for
localization. [An alternative localization approach that we used years ago on
the Agilent Technologies project involved localizing just the pages accessed by
customers -- whereas "administration" pages, such as **Site Settings**, were
kept in English due to requirements from the support team.]

{{< div-block "note" >}}

> **Note**
>
> I believe the SharePoint product team expects you to install language packs
> prior to creating site collections that will contain localized sites. This is
> based on something I recall reading on TechNet.
>
> However, I don't believe this is a realistic expectation -- at least not in my
> experience. Organizations should be able to expand their existing site
> collections to support additional languages over time as the need arises.
>
> The only bug that I'm aware of when installing SharePoint language packs
> _after_ creating the site collection (and subsequently creating localized
> sites) is that some items referenced by the SharePoint pages (e.g. CSS files)
> do not get added to the Style Library. However, you can simply upload these
> files yourself to workaround the issue.

{{< /div-block >}}

One of the first things you'll discover when deploying custom code to a
SharePoint environment that has language packs installed is whether or not your
code accesses SharePoint objects in a "language-agnostic" manner. For example,
consider the following code:

```
SPListItem oListItem = ...

oListItem["Title"] = "My Item";
...
```

```C#
oListItem.Update();
```

You might very well have code like this in your solution today and it has been
running fine for years. However, let's suppose that your solution now attempts
to perform the same operation on a SharePoint site created in the Spanish
language. In this case, the code above will fail because the display name of the
"Title" field is now localized in Spanish.

Fortunately, SharePoint has long provided the
**[SPBuiltInFieldId](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.spbuiltinfieldid%28v=office.12%29.aspx)**
class that makes it easy to reference out-of-the-box fields regardless of the
language of the SharePoint site containing the list item:

```C#
SPListItem oListItem = ...

oListItem[SPBuiltInFieldId.Title] = "My Item";
...
oListItem.Update();
```

[Unfortunately, much of the sample code on MSDN does not show the use of **SPBuiltInFieldId** when referencing "built-in" fields. For example, just this morning I pulled the first code sample above from [http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.splistitem(v=office.12).aspx](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.splistitem%28v=office.12%29.aspx).
Is it any wonder that we SharePoint developers wrote "bad code" from the start
;-)]

Since I tend to work with large enterprise customers deploying Internet-facing
sites with SharePoint, I work a lot with Publishing sites. Consequently, some
time ago I wrote the following code:

```C#
ImageFieldValue pageImage =
    (ImageFieldValue)page.ListItem["Page Image"];

if (pageImage == null)
{
    pageImage = new ImageFieldValue();
}

if (string.IsNullOrEmpty(pageImage.ImageUrl) == true)
{
    pageImage.ImageUrl = defaultImageUrl;
    page.ListItem["Page Image"] = pageImage;
    page.Update();

    ...
```

This code subseqeuntly broke when running against localized SharePoint sites.
Similar to the **SPBuiltInFieldId** class, the SharePoint Publishing
infrastructure provides the
**[FieldId](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.publishing.fieldid.aspx)**
class for identifying fields like "Page Image":

```C#
ImageFieldValue pageImage =
    (ImageFieldValue)page.ListItem[FieldId.PublishingPageImage];

if (pageImage == null)
{
    pageImage = new ImageFieldValue();
}

if (string.IsNullOrEmpty(pageImage.ImageUrl) == true)
{
    pageImage.ImageUrl = defaultImageUrl;
    page.ListItem[FieldId.PublishingPageImage] = pageImage;
    page.Update();
    ...
```

[Personally, I would have preferred this to be `PublishingFieldId.PageImage`
instead, but it is what it is.]

When creating your own content types and custom fields, I recommend following a
similar pattern and providing a class that defines the field IDs that can
subsequently be referenced in code. For example:

```C#
using System;

namespace Fabrikam.Demo.Web
{
    /// <summary>
    /// Provides a set of read-only properties that uniquely identify custom
    /// SharePoint fields.
    /// </summary>
    public static class FabrikamFieldId
    {
        /// <summary>
        /// The unique identifer for the <code>AnnouncementEndDate</code> field.
        /// </summary>
        public static readonly Guid AnnouncementEndDate =
            new Guid("{38AD0629-93B7-4a6b-A7AE-DAE5876C5A10}");

        /// <summary>
        /// The unique identifer for the <code>AnnouncementStartDate</code>
        /// field.
        /// </summary>
        public static readonly Guid AnnouncementStartDate =
            new Guid("{AE9236D0-169F-40a3-8B86-3D8DD4E6B6FE}");

        /// <summary>
        /// The unique identifer for the <code>Height</code> field.
        /// </summary>
        public static readonly Guid Height =
            new Guid("{8EE9FD29-E45C-4888-A763-8EFECAC3910C}");

        /// <summary>
        /// The unique identifer for the <code>Width</code> field.
        /// </summary>
        public static readonly Guid Width =
            new Guid("{63508AE4-E6AF-4850-A194-1EEA35158C1C}");

    }
}
```

The class above provides a convenient way of accessing custom fields defined in
the following Elements.xml file:

```XML
<?xml version="1.0" encoding="utf-8" ?>
<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
  <ContentType
    ID="0x010100C568DB52D9D0A14D9B2FDCC96666E9F2007948130EC3DB064584E219954237AF390064DEA0F50FC8C147B0B6EA0636C4A7D400840BCDB6A21043d4961D1140D1233749"
    Name="$Resources:Fabrikam.Portal.Web, PageLayouts_ContentType_AnnouncementPage_Name"
    Description="$Resources:Fabrikam.Portal.Web, PageLayouts_ContentType_AnnouncementPage_Description"
    Group="$Resources:Fabrikam.Portal.Web, PageLayouts_ContentTypeGroup_FabrikamContentTypes">
    <FieldRefs>
      <FieldRef ID="{AE9236D0-169F-40a3-8B86-3D8DD4E6B6FE}"
        Name="AnnouncementStartDate" />
      <FieldRef ID="{38AD0629-93B7-4a6b-A7AE-DAE5876C5A10}"
        Name="AnnouncementEndDate" />
      <FieldRef ID="{63508AE4-E6AF-4850-A194-1EEA35158C1C}"
        Name="Width" />
      <FieldRef ID="{8EE9FD29-E45C-4888-A763-8EFECAC3910C}"
        Name="Height" />
    </FieldRefs>
    <DocumentTemplate TargetName="/_layouts/CreatePage.aspx" />
  </ContentType>
</Elements>
```
