---
title: Unit tests for filtering errors in ELMAH
date: 2012-02-28T21:42:33-07:00
excerpt:
  In my previous post, I briefly mentioned the unit tests I created while trying
  to figure out why my ELMAH filter was not working as expected. Well, here they
  are for your enjoyment (or idle curiosity).
aliases:
  [
    "/blog/jjameson/archive/2012/02/28/unit-tests-for-filtering-errors-in-elmah.aspx",
  ]
draft: true
categories: ["Development"]
tags: ["Core Development", "Visual Studio", "Web Development"]
attachment: 
  url: "https://assets.technologytoolbox.com/blog/jjameson/Documents/Elmah.DeveloperTests.zip"
  fileName: Elmah.DeveloperTests.zip
  fileSizeInBytes: 4843
---

In
[last night's post](/blog/jjameson/2012/02/28/filter-elmah-email-messages-to-avoid-getting-spammed-by-hackers),
I described how I spent some time this past weekend configuring an ELMAH filter
to avoid getting spammed with email messages as a result of failed attempts to
hack the TechnologyToolbox.com website.

I also noted how I took a short detour during the process to write some unit
tests when my JavaScript filter did not work as expected.

I'm not sure whether these unit tests will prove to be valuable for others (or
to me at some future point in time), but I thought it a good idea to invest a
few minutes creating a blog post about them just in case.

As I mentioned before, I created these unit tests to validate my custom
JavaScript error filter (in other words, the content of the
`errorFilter/test/jscript/expression` element in Web.config). In ELMAH, this
filter is implemented in the **JScriptAssertion** class.

However, I didn't start by creating a unit test for **JScriptAssertion**.
Rather, as I typically do when approaching a problem, I started with a simpler
scenario (so I could develop an approach for the unit tests and verify things
worked as expected).

The first unit test I wrote was for the **TypeAssertion** class:

```C#
        [TestMethod()]
        public void TypeAssertionTest001()
        {
            IContextExpression source = null;

            Type expectedType = typeof(HttpRequestValidationException);
            bool byCompatibility = false;

            const bool expected = false;

            Exception e = new FileNotFoundException();

            var context = new Elmah.ErrorFilterModule.AssertionHelperContext(
                e,
                null);

            TypeAssertion target = new TypeAssertion(
                source,
                expectedType,
                byCompatibility);

            bool actual = target.Test(context);

            Assert.AreEqual(expected, actual);
        }
```

The **TypeAssertion** shown above is equivalent to the following in Web.config:

```XML
  <elmah>
    ...
    <errorFilter>
      <test>
        <is-type binding="BaseException"
          type="System.Web.HttpRequestValidationException, System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" />
      </test>
    </errorFilter>
  </elmah>
```

The test simply verifies this filter does _not_ match a
**FileNotFoundException**.

Then I created a second unit test to verify the filter _does_ match an
**HttpRequestValidationException**:

```C#
        [TestMethod()]
        public void TypeAssertionTest002()
        {
            IContextExpression source = null;

            Type expectedType = typeof(HttpRequestValidationException);
            bool byCompatibility = false;

            const bool expected = true;

            Exception e = new HttpRequestValidationException();

            var context = new Elmah.ErrorFilterModule.AssertionHelperContext(
                e,
                null);

            TypeAssertion target = new TypeAssertion(
                source,
                expectedType,
                byCompatibility);

            bool actual = target.Test(context);

            Assert.AreEqual(expected, actual);
        }
```

The next step was to translate these **TypeAssertion** unit tests into
equivalent unit tests for the **JScriptAssertion** class:

```C#
        [TestMethod()]
        public void JScriptAssertionTest001()
        {
            const string expression =
@"// @assembly mscorlib
// @assembly System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a
// @import System.Web

BaseException instanceof HttpRequestValidationException";

            Exception e = new FileNotFoundException();
            const bool expected = false;

            var context = new Elmah.ErrorFilterModule.AssertionHelperContext(
                e,
                null);

            JScriptAssertion target = new JScriptAssertion(expression);
            bool actual = target.Test(context);

            Assert.AreEqual(expected, actual);
        }

        /// <summary>
        /// Basic test for <see cref="JScriptAssertion.Test"/> method.
        /// </summary>
        [TestMethod()]
        public void JScriptAssertionTest002()
        {
            const string expression =
@"// @assembly mscorlib
// @assembly System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a
// @import System.Web

BaseException instanceof HttpRequestValidationException";

            Exception e = new HttpRequestValidationException();
            const bool expected = true;

            var context = new Elmah.ErrorFilterModule.AssertionHelperContext(
                e,
                null);

            JScriptAssertion target = new JScriptAssertion(expression);
            bool actual = target.Test(context);

            Assert.AreEqual(expected, actual);
        }
```

From there, I simply kept "iterating" the JavaScript filter being tested until
it resembled the filter that I was having trouble with:

```C#
        [TestMethod()]
        public void JScriptAssertionTest003()
        {
            const string expression =
@"// @assembly mscorlib
// @assembly System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a
// @import System.Web

// Do not send email notification...
FilterSourceType.Name == 'ErrorMailModule'
// ...when hackers attempt something like 'http://.../?<script>'
&& BaseException instanceof HttpRequestValidationException";

            Exception e = new HttpRequestValidationException();
            const bool expected = false;

            var context = new Elmah.ErrorFilterModule.AssertionHelperContext(
                e,
                null);

            JScriptAssertion target = new JScriptAssertion(expression);
            bool actual = target.Test(context);

            Assert.AreEqual(expected, actual);
        }

        [TestMethod()]
        public void JScriptAssertionTest004()
        {
            const string expression =
@"// @assembly mscorlib
// @assembly System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a
// @import System.Web

// Do not send email notification...
FilterSourceType.Name == 'ErrorMailModule'
// ...when hackers attempt something like 'http://.../?<script>'
&& BaseException instanceof HttpRequestValidationException";

            Exception e = new HttpRequestValidationException();
            const bool expected = true;

            var module = new Elmah.ErrorMailModule();

            var context = new Elmah.ErrorFilterModule.AssertionHelperContext(
                module,
                e,
                null);

            JScriptAssertion target = new JScriptAssertion(expression);
            bool actual = target.Test(context);

            Assert.AreEqual(expected, actual);
        }

        [TestMethod()]
        public void JScriptAssertionTest005()
        {
            const string expression =
@"// @assembly mscorlib
// @assembly System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a
// @import System.Web

// Do not send email notification...
FilterSourceType.Name == 'ErrorMailModule'
&& (
// ...when hackers attempt something like 'http://.../?<script>'
BaseException instanceof HttpRequestValidationException
// ...or when someone attempts to hack view state
|| Exception.Message ==
    'The state information is invalid for this page and might be corrupted.'
// ...or when someone attempts to hack the site using the
// 'padding oracle exploit'
// (http://technet.microsoft.com/en-us/security/advisory/2416728)
|| Exception.Message ==
    'This is an invalid script resource request.'
|| Exception.Message ==
    'This is an invalid webresource request.')";

            Exception e = new HttpRequestValidationException();
            const bool expected = true;

            var module = new Elmah.ErrorMailModule();

            var context = new Elmah.ErrorFilterModule.AssertionHelperContext(
                module,
                e,
                null);

            JScriptAssertion target = new JScriptAssertion(expression);
            bool actual = target.Test(context);

            Assert.AreEqual(expected, actual);
        }

        [TestMethod()]
        public void JScriptAssertionTest006()
        {
            const string expression =
@"// @assembly mscorlib
// @assembly System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a
// @import System.Web

// Do not send email notification when hackers attempt something
// like 'http://.../?<script>'
(FilterSourceType.Name == 'ErrorMailModule'
&& BaseException instanceof HttpRequestValidationException)
// Do not send email notification when someone attempts to hack
// view state
|| (FilterSourceType.Name == 'ErrorMailModule'
&& Exception.Message ==
    'The state information is invalid for this page and might be corrupted.')
// Do not send email notification when someone attempts to hack
// the site using the 'padding oracle exploit'
// (http://technet.microsoft.com/en-us/security/advisory/2416728)
|| (FilterSourceType.Name == 'ErrorMailModule'
&& (Exception.Message ==
    'This is an invalid script resource request.'
    || Exception.Message ==
        'This is an invalid webresource request.'))";

            Exception e = new HttpException("This is an invalid webresource request.");
            const bool expected = true;

            var module = new Elmah.ErrorMailModule();

            var context = new Elmah.ErrorFilterModule.AssertionHelperContext(
                module,
                e,
                null);

            JScriptAssertion target = new JScriptAssertion(expression);
            bool actual = target.Test(context);

            Assert.AreEqual(expected, actual);
        }

        [TestMethod()]
        public void JScriptAssertionTest007()
        {
            const string expression =
@"// @assembly mscorlib
// @assembly System.Web, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a
// @import System.Web

// Do not send email notification...
FilterSourceType.Name == 'ErrorMailModule'
&& (
// ...when hackers attempt something like 'http://.../?<script>'
BaseException instanceof HttpRequestValidationException
// ...or when someone attempts to hack view state
|| Exception.Message ==
    'The state information is invalid for this page and might be corrupted.'
// ...or when someone attempts to hack the site using the
// " + "\"padding oracle exploit\"" + @"
// (http://technet.microsoft.com/en-us/security/advisory/2416728)
|| Exception.Message ==
    'This is an invalid script resource request.'
|| Exception.Message ==
    'This is an invalid webresource request.')";

            Exception e = new HttpException("This is an invalid webresource request.");
            const bool expected = true;

            var module = new Elmah.ErrorMailModule();

            var context = new Elmah.ErrorFilterModule.AssertionHelperContext(
                module,
                e,
                null);

            JScriptAssertion target = new JScriptAssertion(expression);
            bool actual = target.Test(context);

            Assert.AreEqual(expected, actual);
        }
    }
```

I was a little baffled when this final unit test came up "green" (since I had
copied and pasted the JavaScript from the Web.config file). As noted in my
previous post, the problem I encountered with the JavaScript filter turned out
to be some sort of caching issue in the **FullTrustEvaluationStrategy** class --
and therefore not something I discovered via the unit tests.

However, the unit tests I created for **JScriptAssertion** reassured me that I
wasn't simply losing my mind when trying to figure out why the JavaScript filter
wasn't working as expected.
