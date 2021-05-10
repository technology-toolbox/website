---
title: Testing for Expected Exceptions with Visual Studio
date: 2007-03-22T07:28:00-06:00
description:
  "Since I seem to be on a roll blogging this morning, I thought I'd see if I
  could get one more in before my baby girl wakes up to have breakfast with
  \"Da-da.\" When transitioning a couple of years ago from using NUnit to Visual
  Studio 2005, I noticed..."
aliases:
  [
    "/blog/jjameson/archive/2007/03/21/testing-for-expected-exceptions-with-visual-studio.aspx",
    "/blog/jjameson/archive/2007/03/22/testing-for-expected-exceptions-with-visual-studio.aspx",
  ]
categories: ["Development"]
tags: ["Core Development"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2007/03/22/testing-for-expected-exceptions-with-visual-studio.aspx"
---

Since I seem to be on a roll blogging this morning, I thought I'd see if I could
get one more in before my baby girl wakes up to have breakfast with "Da-da."

When transitioning a couple of years ago from using NUnit to Visual Studio 2005,
I noticed that the ExpectedExceptionAttribute didn't quite do what I expected
(no pun intended) based on my experience with NUnit.

With NUnit, you could specify the expected message text of the exception and
NUnit would transparently perform an assertion on the actual message. In Visual
Studio 2005, I found the following "pattern" useful for mimicking the familiar
behavior of NUnit:

```C#
[TestMethod()]
[ExpectedException(typeof(ArgumentException))]
public void FindByWhidWithInvalidWhid()
{
    const string expectedExceptionMessage = "A valid WHID must be specified.\r\nParameter name: whid";

    string project1Url = Properties.Settings.Default.Project1Url;

    SPSite site = new SPSite(project1Url);

    try
    {
        PrimaryDocumentService.FindByWhid(site, 0);
    }
    catch (ArgumentException ex)
    {
        Assert.AreEqual(expectedExceptionMessage, ex.Message);
        throw;
    }
}
```

The internal try/catch block validates the expected message, whereas the
`ExpectedException` attribute on the method delegates the remaining work to the
Visual Studio test manager (i.e. showing green vs. red for each test).
