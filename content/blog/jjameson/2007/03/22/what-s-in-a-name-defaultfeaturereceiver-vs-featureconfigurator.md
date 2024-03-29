---
title: "What's in a name? \"DefaultFeatureReceiver\" vs. \"FeatureConfigurator\""
date: 2007-03-22T07:07:00-06:00
description:
  In my previous post ( Scope Dependencies for SharePoint Features ) you may
  have noticed that in the Feature.xml file, I specified the feature receiver
  class as DefaultFeatureReceiver but the code sample is actually from
  FeatureConfigurator . This warrants...
aliases:
  [
    "/blog/jjameson/archive/2007/03/21/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator.aspx",
    "/blog/jjameson/archive/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator.aspx",
  ]
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/03/22/what-s-in-a-name-defaultfeaturereceiver-vs-featureconfigurator.aspx"
---

In my previous post
([Scope Dependencies for SharePoint Features](/blog/jjameson/2007/03/22/scope-dependencies-for-sharepoint-features))
you may have noticed that in the Feature.xml file, I specified the feature
receiver class as **DefaultFeatureReceiver** but the code sample is actually
from **FeatureConfigurator**. This warrants a little explanation.

In order to simplify the development and debugging of feature receivers, I find
it much easier to put the bulk of the code in a FeatureConfigurator class and
then have the DefaultFeatureReceiver class simply be a "thin shell" that
utilizes the underlying FeatureConfigurator class to do the bulk of the work:

```C#
namespace Fabrikam.Project1.PublicationLibrary.Configuration
{
    [CLSCompliant(false)]
    public class DefaultFeatureReceiver : SPFeatureReceiver
      {
        [SharePointPermission(SecurityAction.LinkDemand, ObjectModel = true)]
        public override void FeatureActivated(
            SPFeatureReceiverProperties properties)
        {
            if (properties == null)
            {
                throw new ArgumentNullException("properties");
            }

            SPWebApplication webApp = properties.Feature.Parent as SPWebApplication;

            if (webApp == null)
            {
                throw new InvalidOperationException(
                    "The feature must be activated at the 'WebApplication' scope.");
            }

            FeatureConfigurator.Configure(webApp);
        }
        ...
   }
}
```

The bulk of the development (and debugging) of the feature receiver is then
performed using a separate Visual Studio test project:

```C#
namespace Fabrikam.Project1.PublicationLibrary.DeveloperTests.Configuration
{
    [TestClass()]
    public class FeatureConfiguratorTest
    {
        ...

        [TestMethod()]
        public void ConfigureTest()
        {
            string project1Url = Properties.Settings.Default.Project1Url;

            Uri webAppUri = new Uri(project1Url);

            SPWebApplication webApp = SPWebApplication.Lookup(webAppUri);

            FeatureConfigurator.Configure(webApp);

            // If we make it to here without an exception, then all is well
            Assert.IsTrue(true);
        }
    }
}
```
