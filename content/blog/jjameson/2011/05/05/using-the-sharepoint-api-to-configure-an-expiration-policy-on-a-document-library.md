---
title: "Using the SharePoint API to Configure an Expiration Policy on a Document Library"
date: 2011-05-05T00:58:00-07:00
excerpt: "While it typically takes less than a minute or two to configure an expiration policy on a SharePoint document library, there may still be reasons why you want to do this using the SharePoint object model instead. 
 For example, suppose I have a \"Temporary..."
aliases: ["/blog/jjameson/archive/2011/05/05/using-the-sharepoint-api-to-configure-an-expiration-policy-on-a-document-library.aspx"]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "SharePoint 2010"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/05/05/using-the-sharepoint-api-to-configure-an-expiration-policy-on-a-document-library.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/05/05/using-the-sharepoint-api-to-configure-an-expiration-policy-on-a-document-library.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

While it typically takes less than a minute or two to configure an expiration policy on a SharePoint document library, there may still be reasons why you want to do this using the SharePoint object model instead.

For example, suppose I have a "Temporary Files" document library for which I want the files to be deleted approximately one day after they are added to the library.

The following C# code shows how to configure an expiration policy on the specified document library (passed as an **SPList** object):

```
using Microsoft.Office.RecordsManagement.InformationPolicy;
using Microsoft.SharePoint;
...
        private const string expirationPolicyFeatureId =
            "Microsoft.Office.RecordsManagement.PolicyFeatures.Expiration";

        private static void ConfigureExpirationPolicy(
            SPList tempFilesLibrary)
        {
            Debug.Assert(tempFilesLibrary != null);

            // HACK: Refresh the SPList object in order to avoid
            // the following error when calling Policy.CreatePolicy:
            //
            // "The object has been updated by another user since it was last
            // fetched."
            SPList tempFilesLibrary2 =
                tempFilesLibrary.ParentWeb.Lists[tempFilesLibrary.ID];
            
            SPContentTypeId documentContentTypeId =
                tempFilesLibrary2.ContentTypes.BestMatch(
                    SPBuiltInContentTypeId.Document);

            SPContentType documentContentType =
                tempFilesLibrary2.ContentTypes[documentContentTypeId];

            Debug.Assert(documentContentType != null);
            
            Policy policy = Policy.GetPolicy(documentContentType);

            if (policy == null)
            {
                Policy.CreatePolicy(documentContentType, null);
                policy = Policy.GetPolicy(documentContentType);
            }

            if (policy.Items[expirationPolicyFeatureId] == null)
            {
                string customData =
"<data>"
    + "<formula id=\"Microsoft.Office.RecordsManagement.PolicyFeatures.Expiration.Formula.BuiltIn\">"
        + "<number>1</number>"
        + "<property>Created</property>"
        + "<period>days</period>"
    + "</formula>"
    + "<action type=\"action\" id=\"Microsoft.Office.RecordsManagement.PolicyFeatures.Expiration.Action.MoveToRecycleBin\" />"
+ "</data>";

                policy.Items.Add(expirationPolicyFeatureId, customData);
            }
        }
```

This code adds an expiration policy to the library if it hasn't yet been configured. If an expiration policy *has* been configured on the document library, then it it ignored (because the policy may have been modified by an administrator -- for example, to increase the time period used to determine which files to delete).

> **Note**
>
> As shown in the HACK comment above, I fetch a new **SPList** object in order to avoid the following error when configuring the expiration policy immediately after creating a new document library:
>
> The object has been updated by another user since it was last fetched.
>
> This error occurred in Microsoft Office SharePoint Server (MOSS) 2007 -- although I just ran a quick test in SharePoint 2010 and it looks like this may no longer be an issue in the new version.

In my next post, I'll explain more about storing files temporarily in SharePoint.

