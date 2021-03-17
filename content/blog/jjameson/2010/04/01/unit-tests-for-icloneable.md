---
title: "Unit Tests for ICloneable"
date: 2010-04-01T06:43:00-06:00
excerpt:
  "A few years ago I developed a class ( SharePointSearchUrlBuilder ) for
  working with SharePoint Search URLs. The class is used to easily build or
  parse the various query string parameters used by SharePoint Search (e.g.
  keywords, search scope, additional..."
aliases: ["/blog/jjameson/archive/2010/03/31/unit-tests-for-icloneable.aspx", "/blog/jjameson/archive/2010/04/01/unit-tests-for-icloneable.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/01/unit-tests-for-icloneable.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/01/unit-tests-for-icloneable.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

A few years ago I developed a class (**SharePointSearchUrlBuilder**) for working
with SharePoint Search URLs. The class is used to easily build or parse the
various query string parameters used by SharePoint Search (e.g. keywords, search
scope, additional query terms, etc.) and serves as the foundation for the
[faceted search](/blog/jjameson/2009/09/18/faceted-search-in-moss-2007-and-the-mssdocprops-issue)
solution I first built for Agilent Technologies.

A common scenario in faceted search is to start with a search results URL and
subsequently manipulate it in a multitude of ways to generate links to related
search results.

To support this scenario, **SharePointSearchUrlBuilder** implements the
**[ICloneable](http://msdn.microsoft.com/en-us/library/system.icloneable.aspx)**
interface. In other words, an instance of **SharePointSearchUrlBuilder** can be
created using the current URL of the search results page and subsequently cloned
multiple times to add or change the search criteria in order to generate links
to view related search results.

Note that the
**[ICloneable.Clone](http://msdn.microsoft.com/en-us/library/system.icloneable.clone.aspx)**
method is defined rather ambiguously. Here are the remarks from the
corresponding documentation on MSDN:

{{< blockquote "font-italic" >}}

Clone can be implemented either as a deep copy or a shallow copy. In a deep
copy, all objects are duplicated; whereas, in a shallow copy, only the top-level
objects are duplicated and the lower levels contain references.

{{< /blockquote >}}

In my mind, a cloned object should *always* be a deep copy. A shallow copy would
most likely lead to subtle bugs when the code that makes a copy doesn't
"realize" it's a shallow copy.

To ensure a class implements the **Clone** method as I expect, I first create a
simple unit test that clones an object and then verifies that all of the members
are still equal:

```
        /// <summary>
        /// Validates that an object is cloned as expected.
        /// </summary>
        [TestMethod()]
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Usage",
            "CA2234:PassSystemUriObjectsInsteadOfStrings")]
        public void Clone001()
        {
            const string searchUrl =
                "/en-US/Search/Pages/default.aspx"
                    + "?k=purification"
                    + "&s=English+(U.S.)+Content"
                    + "&a=ContentTypeName:Brochure";

            const string expectedUrl = searchUrl;
            const string expectedPath =
                "/en-US/Search/Pages/default.aspx";

            const string expectedScope = "English (U.S.) Content";
            const string expectedKeywords = "purification";
            const string expectedAdditionalQueryTerms =
                "ContentTypeName:Brochure";

            SharePointSearchUrlBuilder urlBuilder =
                new SharePointSearchUrlBuilder(searchUrl);

            SharePointSearchUrlBuilder clone =
                (SharePointSearchUrlBuilder)urlBuilder.Clone();

            string actualUrl = clone.ToString();

            Assert.AreNotSame(urlBuilder, clone);
            Assert.AreEqual(expectedUrl, actualUrl);
            Assert.AreEqual(expectedPath, clone.Path);
            Assert.AreEqual(expectedKeywords, clone.Keywords);
            Assert.AreEqual(expectedScope, clone.Scope);
            Assert.AreEqual(expectedAdditionalQueryTerms,
                clone.AdditionalQueryTerms);

        }
```

Next, I create a unit test that clones an object, subsequently changes the
original object, and then verifies that the cloned object is not modified:

```
        /// <summary>
        /// Validates that a deep copy is made when an object is cloned.
        /// </summary>
        [TestMethod()]
        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Usage",
            "CA2234:PassSystemUriObjectsInsteadOfStrings")]
        public void Clone002()
        {
            const string searchUrl =
                "/en-US/Search/Pages/default.aspx"
                    + "?k=purification"
                    + "&s=English+(U.S.)+Content"
                    + "&a=ContentTypeName:Brochure";

            const string expectedUrl = searchUrl;
            const string expectedPath =
                "/en-US/Search/Pages/default.aspx";

            const string expectedScope = "English (U.S.) Content";
            const string expectedKeywords = "purification";
            const string expectedAdditionalQueryTerms =
                "ContentTypeName:Brochure";

            SharePointSearchUrlBuilder urlBuilder =
                new SharePointSearchUrlBuilder(searchUrl);

            SharePointSearchUrlBuilder clone =
                (SharePointSearchUrlBuilder)urlBuilder.Clone();

            urlBuilder.Keywords = "purificación";
            urlBuilder.Scope = "Spanish Content";
            urlBuilder.AdditionalQueryTerms = "ContentTypeName:Manual";
            urlBuilder.AddPropertyFilter(
                "ContentTypeName",
                "MSDS");

            string actualUrl = clone.ToString();

            Assert.AreNotSame(urlBuilder, clone);
            Assert.AreEqual(expectedUrl, actualUrl);
            Assert.AreEqual(expectedPath, clone.Path);
            Assert.AreEqual(expectedKeywords, clone.Keywords);
            Assert.AreEqual(expectedScope, clone.Scope);
            Assert.AreEqual(expectedAdditionalQueryTerms,
                clone.AdditionalQueryTerms);

        }
```

Note that the second unit test makes the calls to `Assert.AreNotSame`
superfluous (if the two object references referred to the same object, then you
obviously couldn't make changes to one without affecting the other).
Nevertheless, I tend to keep them in there anyway.

I also try to make sure that I explicitly document that a deep copy is made when
cloning an object, as shown in the following example for the
**SharePointSearchUrlBuilder** class:

```
        /// <summary>
        /// Creates a new object that is a deep copy of the current instance.
        /// </summary>
        /// <returns>A new object that is a deep copy of this instance.</returns>
        public object Clone()
        {
            SharePointSearchUrlBuilder clone =
                new SharePointSearchUrlBuilder();

            clone.cachedSearchUrl = this.cachedSearchUrl;
            clone.path = this.path;

            if (this.queryStringParams != null)
            {
                clone.queryStringParams =
                    new Dictionary<string, string>(this.queryStringParams);
            }

            return clone;
        }
```
