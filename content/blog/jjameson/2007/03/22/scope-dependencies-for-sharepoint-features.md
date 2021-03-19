---
title: Scope Dependencies for SharePoint Features
date: 2007-03-22T06:17:00-06:00
excerpt:
  While integrating various SharePoint features last week, I discovered some of
  the details around the dependency rules when trying to associate one feature
  to another. In our solution, we have created a feature for specifying custom
  fields (i.e. columns...
aliases:
  [
    "/blog/jjameson/archive/2007/03/21/scope-dependencies-for-sharepoint-features.aspx",
    "/blog/jjameson/archive/2007/03/22/scope-dependencies-for-sharepoint-features.aspx",
  ]
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/03/22/scope-dependencies-for-sharepoint-features.aspx"
---

While integrating various SharePoint features last week, I discovered some of
the details around the dependency rules when trying to associate one feature to
another.

In our solution, we have created a feature for specifying custom fields (i.e.
columns) and content types. This feature is called the **{Fabrikam Project1}
Publication Content Types** [company and project name replaced to protect the
innocent ;-) ]. This makes it very easy to setup a new environment for our
solution (e.g. DEV to TEST to PROD).

We have a different feature that builds out a custom version of the Document
Center (a.k.a. the BDR or Business Document Repository), for example, to remove
the default **Documents** doc lib, create a bunch of other doc libs instead,
enable versioning on the doc libs, enable content types, set the default content
type appropriately based on the particular doc lib, yadda, yadda, yadda.

This other feature is called **{Fabrikam Project1} Publication Library**, and
naturally it has a dependency on the **{Fabrikam Project1} Publication Content
Types** feature.

Originally, I had both features scoped to **Site** and setup a dependency from
the **Publication Library** feature to the **Publication Content Types**
feature:

```
<?xml version="1.0" encoding="utf-8"?>
<Feature xmlns="http://schemas.microsoft.com/sharepoint/"
  Id="49B204D0-7E35-4460-A691-A7D481C463B4"
  ...
  Scope="Site"
  ...
  ReceiverAssembly="Fabrikam.Project1.PublicationLibrary, Version=1.0.0.0, Culture=neutral, PublicKeyToken=8b7a42e9b9b5355f"
  ReceiverClass="Fabrikam.Project1.PublicationLibrary.Configuration.DefaultFeatureReceiver">
  <ActivationDependencies>
    <!-- Fabrikam.Project1.PublicationContentTypes -->
    <ActivationDependency FeatureId="9F5C14F1-CF58-47c7-BBBA-DA9A8637DEAB" />
  </ActivationDependencies>
  <ElementManifests>
  </ElementManifests>
</Feature>
```

I later realized that the **Publication Library** feature would be better scoped
to **WebApplication** instead of **Site** (perhaps I'll blog about the reasons
for this at another time), so I deactivated and uninstalled the feature, made a
quick change to the Feature.xml file to change the scope, and then attempted to
install and activate the updated feature. Unfortunately things did not go quite
as smoothly as I had expected...

{{< console-block-start >}}

C:\NotBackedUp\Fabrikam\Project1\Main\PublicationLibrary\DeploymentFiles\Scripts&gt;{{<
kbd "\"Activate Feature.cmd\"" >}}

{{< sample-block >}}

Activating Fabrikam.Project1.PublicationLibrary on url - http://project1-local

Dependency feature 'Fabrikam.Project1.PublicationContentTypes' (id:
9f5c14f1-cf58-47c7-bbba-da9a8637deab) is not properly scoped for feature
'Fabrikam.Project1.PublicationLibrary' (id:
49b204d0-7e35-4460-a691-a7d481c463b4). Its scope 'Site' must be equal to or
higher than 'WebApplication'.

{{< /sample-block >}}

{{< console-block-end >}}

Ouch...okay, no problem, I guess I'll just change the **Publication Content
Types** feature to be scoped to **WebApplication** as well (instead of
**Site**). Another quick deactivate, uninstall, XML file tweak, deploy, install,
activate (I say "quick" only because of the scripts that we have to simplify the
deployment) and...

{{< console-block-start >}}

C:\NotBackedUp\Fabrikam\Project1\Main\PublicationContentTypes\DeploymentFiles\Scripts&gt;{{<
kbd "\"Install Feature.cmd\"" >}}

{{< sample-block >}}

Installing Fabrikam.Project1.PublicationContentTypes

Elements of type 'Field' are not supported at the 'WebApplication' scope. This
feature could not be installed.

{{< /sample-block >}}

{{< console-block-end >}}

Ugh...

Well, so if I want the **Publication Library** feature to be installed with
**WebApplication** scope, then I have no choice but to remove the dependency on
the **Publication Content Types** feature. However, there really is a hard
dependency on the other feature, so I really hate giving that up. After a couple
of minutes, it came to me...

We already had a feature receiver for the **Publication Library** feature (as
mentioned earlier to create and configure the Document Center). Why not just add
a dependency check in there? Eureka!

```
namespace Fabrikam.Project1.PublicationLibrary.Configuration
{
    [CLSCompliant(false)]
    public sealed class FeatureConfigurator
    {
        readonly static Guid publicationContentTypesFeatureId =
            new Guid("9F5C14F1-CF58-47c7-BBBA-DA9A8637DEAB");

        private FeatureConfigurator() { } // all methods are static

        public static void Configure(
            SPWebApplication webApp)
        {
            if (webApp == null)
            {
                throw new ArgumentNullException("webApp");
            }

            Debug.WriteLine("Configuring Fabrikam.Project1.PublicationLibrary feature...");

            SPSite site = webApp.Sites["/"];

            // We cannot specify an explicit dependency on the PublicationContentTypes feature
            // because that feature must be scoped to 'Site' (since it contains Field elements).
            // Therefore we check this dependency via custom code when this feature is activated.
            SPFeature feature = site.Features[publicationContentTypesFeatureId];

            if (feature == null)
            {
                throw new InvalidOperationException(
                    "The Publication Content Types feature must be activated on the site"
                    + " in order to use the Publication Library feature.");
            }

            ...
        }
    }
}
```

Good enough for me!
