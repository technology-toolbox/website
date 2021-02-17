---
title: "Test Driven Development (TDD) in the Real World, Part 2"
date: 2010-04-09T03:48:00-07:00
excerpt: "In part 
1 of this post, I provided my high-level thoughts on doing Test Driven Development 
(TDD) in the real world, but I didn&#39;t get around to walking through an actual sample.
To start off simple (but still real world), let&#39;s imagine we have a scenario where we need to truncate 
a string to a limited number of characters for display or output purposes. However, 
instead of just chopping off the string at the specified number of characters, we 
want to apply a little &quot;intell"
aliases: ["/blog/jjameson/archive/2010/04/08/test-driven-development-tdd-in-the-real-world-part-2.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/09/test-driven-development-tdd-in-the-real-world-part-2.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/09/test-driven-development-tdd-in-the-real-world-part-2.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog
> ever goes away.

In [part
1 of this post](/blog/jjameson/2010/04/08/tdd-in-the-real-world-part-1), I provided my high-level thoughts on doing Test Driven Development  (TDD) in the real world, but I didn't get around to walking through an actual sample.

To start off simple (but still real world), let's imagine we have a scenario  where we need to truncate a string to a limited number of characters for display  or output purposes. However, instead of just chopping off the string at the specified  number of characters, we want to apply a little "intelligence" -- such as trying  to break on complete words, and adding an ellipsis (...) to the end.

For example, if we pass in `"Some really long string with lots of characters"`  and specify to truncate the string to 15 characters, then we should get back {{< sample-output "\"Some really...\"" >}}.

[Note that if you are trying to conserve real estate on a Web page, there is  actually [a
much better way of doing this with CSS](/blog/jjameson/2009/11/01/constraining-tables-with-css), so please don't think of this scenario  in that context.]

Applying the principles of TDD, we know we should:

1. Write the unit tests first.
2. Add the minimal amount of code necessary to get the unit tests to compile.
3. Run the tests and ensure they fail ("red").
4. Write the code to make the unit tests pass ("green").
5. Repeat steps 1-4 until you consider the feature "done." (Note that you should
   also consider checking in your code each time you successfully complete step
   4.)

So for this simple scenario, what unit test(s) should we write first?

Let's say we want to implement this functionality via the **StringHelper.Truncate**  method. In order to truncate a string, we need to pass an input string as well as  the maximum length of the truncated string that gets returned.

Some people might start out by writing just one unit test (for example, passing  in a long string like the sample above and ensuring it gets truncated as expected).  Personally, I often create several unit tests right away in order to capture keys  aspects of the behavior I am trying to implement. However, I often like to start  off as simple as possible -- even with something that seems as trivial as a method  to truncate a string.

One of the things that I love about TDD is it gets me thinking right away about  the "happy path" scenario as well as what should happen when, for example, a parameter  is null or invalid.

Here is what comes to immediately to mind for the **StringHelper.Truncate**  method:

- How should the method behave if a null string is specified?
- What are the limits on the maximum length parameter?

More often than not, when a parameter is null, you simply throw an **ArgumentNullException** and be done with it -- but is that the right behavior in this case? In  my opinion, the answer is no -- if we specify a null or empty string, then the **StringHelper.Truncate** method should should simply return a null  or empty string, respectively.

From this foundation, let's start with the following unit tests:

```
/// <summary>
        /// Validates that a null input string returns null.
        /// </summary>
        [TestMethod()]
        public void Truncate001()
        {
            const string input = null;
            const int maxLength = 80;
            const string expected = null;

            string actual = StringHelper.Truncate(input, maxLength);

            Assert.AreEqual(expected, actual);
        }

        /// <summary>
        /// Validates that an empty input string returns an empty string.
        /// </summary>
        [TestMethod()]
        public void Truncate002()
        {
            string input = string.Empty;
            const int maxLength = 80;
            string expected = string.Empty;

            string actual = StringHelper.Truncate(input, maxLength);

            Assert.AreEqual(expected, actual);
        }
```

You might have written the first unit test slightly different -- perhaps using  the `Assert.IsNull` method instead,  but it doesn't really matter (at least not to me), or perhaps you don't use `const`as aggressively as I do.  [I think that must be the ol' C++ developer in me that likes to easily catch myself  doing something "stupid" in code.]

You also might tend to name your unit tests with something more meaningful than `Truncate001` and `Truncate002`. Personally, most of the time  I don't like to expend the effort to figure out what to name my test methods.

Some people might say that these arbitrary method names don't help you quickly  understand what went wrong when a unit test fails, but to be honest, `IDontBotherTryingToDecipherTheReasonForTheFailureFromTheUnitTestName`  ;-)

Instead, when a unit test goes from "green" to "red" I jump straight to the source  of the unit test to figure out what the unit test is validating in order to understand  why it's no longer passing. If you don't mind coming up with lengthy method names  for your unit tests, then by all means, go right ahead.

I should also point out how I like to organize my unit tests. Let's suppose that  the **StringHelper** class will reside in the **Fabrikam.Demo.CoreServices** assembly. In addition to the **Fabrikam.Demo.CoreServices** project file, I create a second project named **Fabrikam.Demo.CoreServices.DeveloperTests**.  Note that as I mentioned in the previous post, I like to use the "DeveloperTests"  moniker to denote that these are tests created by the Development team (as opposed  to the Test team) and may contain both unit tests as well as integration tests (where  it makes more sense to create an integration test instead of -- or in addition to  -- a unit test).

Okay, let's get back to writing the unit tests...

While it seems obvious that the maximum length parameter must be greater than  zero, it's also important to realize that if we were to specify a value of 1 or  2, then we wouldn't even have enough space for the ellipsis (...) in the returned  string.

In order to validate the maximum length parameter, let's assume that it must  be a minimum of 4 (to allow for a single character from the input string and the  three characters for the ellipsis). This is definitely a somewhat arbitrary number  (and arguably not an adequate length) -- and you might very well have chosen a minimum  of 5, 10, or perhaps something even larger. However, in order to make the **StringHelper.Truncate** method as flexible as possible, let's use  a minimum of 4.

From this we can write another unit test right away:

```
/// <summary>
        /// Validates that an exception is thrown when maxLength is less than 4.
        /// </summary>
        [TestMethod()]
        [ExpectedException(typeof(ArgumentException))]
        public void TruncateInvalidParameter001()
        {
            const string expectedExceptionMessage =
                "A string cannot be truncated to less than 4 characters."
                + "\r\nParameter name: maxLength";

            const string input = null;
            const int maxLength = 3;

            try
            {
                string actual = StringHelper.Truncate(input, maxLength);
            }
            catch (ArgumentException ex)
            {
                Assert.AreEqual(expectedExceptionMessage, ex.Message);
                throw;
            }
        }
```

If this unit test doesn't make sense, take a look at my previous post on [testing for expected exceptions in Visual Studio](/blog/jjameson/2007/03/22/testing-for-expected-exceptions-with-visual-studio).

At this point, it seems like a good point to advance from TDD step 1 ("Write  the unit tests first") to step 2 ("Add the minimal amount of code necessary to get  the unit tests to compile").

In order to get the unit tests to compile, we need to stub out the **StringHelper.Truncate** method:

```
public static class StringHelper
    {
        public static string Truncate(
            string input,
            int maxLength)
        {
            return "TODO:";
        }
    }
}
```

Unfortunately, this isn't quite enough to successfully compile the code -- assuming  you've enabled code analysis on the project (which is [something I always recommend doing](/blog/jjameson/2009/10/31/recommendations-for-code-analysis)) -- due to the following errors:

{{< blockquote "font-italic text-danger" >}}

error : CA1801 : Microsoft.Usage : Parameter 'input' of 'StringHelper.Truncate(string,
int)' is never used. Remove the parameter or use it in the method body.

error : CA1801 : Microsoft.Usage : Parameter 'maxLength' of 'StringHelper.Truncate(string,
int)' is never used. Remove the parameter or use it in the method body.

{{< /blockquote >}}

To resolve these errors, let's add a little more code to the **Truncate** method:

```
public static string Truncate(
            string input,
            int maxLength)
        {
            Debug.Assert(input != null);
            Debug.Assert(maxLength > 0);

            return "TODO:";
        }
```

Note that for `public`methods,  I really don't like to see `Debug.Assert`being used to validate the parameters (on the other hand, this is actually  what I prefer to be used in `private`methods). We'll fix that in a minute.

However, we still encounter one more error when trying to compile the DeveloperTests  project:

{{< blockquote "font-italic text-danger" >}}

error : CA1804 : Microsoft.Performance : 'StringHelperTest.TruncateInvalidParameter001()' declares a variable, 'actual', of type 'string', which is never used or is only assigned to. Use this variable or remove it.

{{< /blockquote >}}

To resolve this error, we need to tweak the unit test a little bit (to remove  the variable used to store the truncated string):

```
/// <summary>
        /// Validates that an exception is thrown when maxLength is less than 4.
        /// </summary>
        [TestMethod()]
        [ExpectedException(typeof(ArgumentException))]
        public void TruncateInvalidParameter001()
        {
            const string expectedExceptionMessage =
                "A string cannot be truncated to less than 4 characters."
                + "\r\nParameter name: maxLength";

            const string input = null;
            const int maxLength = 3;

            try
            {
                // Prevent code analysis warning (CA1804) by not assigning
                // the return value to a local variable.
                StringHelper.Truncate(input, maxLength);
            }
            catch (ArgumentException ex)
            {
                Assert.AreEqual(expectedExceptionMessage, ex.Message);
                throw;
            }
        }
```

At this point, the solution builds without any errors and we can advance to TDD  step 3 ("Run the tests and ensure they fail").

When the tests are run, we get a couple of debug assertion failures, but if we  click the **Ignore** button a couple of times we find that all three  unit tests are failing (which is exactly what we want at this point).

Now let's move on to TDD step 4 ("Write the code to make the unit tests pass").  Let's replace the debug assertions with code that actually does what we need it  to do:

```
public static string Truncate(
            string input,
            int maxLength)
        {
            if (maxLength < 4)
            {
                throw new ArgumentException(
                    "A string cannot be truncated to less than 4 characters.",
                    "maxLength");
            }

            if (string.IsNullOrEmpty(input))
            {
                return input;
            }

            return "TODO:";
        }
```

Running the unit tests again, we find that all of the unit tests pass. Woohoo!

However, we're obviously not done because the **Truncate** method  clearly still has a ways to go before it actually does something useful.

Let's add another unit test by copying and pasting **Truncate001** and modifying it accordingly:

```
/// <summary>
        /// Validates that an input string with a space is truncated as
        /// expected.
        /// </summary>
        [TestMethod()]
        public void Truncate003()
        {
            const string input = "foo bar";
            const int maxLength = 6;
            const string expected = "foo...";

            string actual = StringHelper.Truncate(input, maxLength);

            Assert.AreEqual(expected, actual);
        }
```

Running the tests confirms that **Truncate003** fails as expected  (since at this point, we are just returning `"TODO:"`  from the **Truncate** method). Let's add a little bit of code to find  the last space in the input string and truncate accordingly:

```
public static string Truncate(
            string input,
            int maxLength)
        {
            if (maxLength < 4)
            {
                throw new ArgumentException(
                    "A string cannot be truncated to less than 4 characters.",
                    "maxLength");
            }

            if (string.IsNullOrEmpty(input))
            {
                return input;
            }

            int index = input.LastIndexOf(' ', maxLength - 3);

            string truncatedText = input.Substring(0, index) + "...";

            return truncatedText;
        }
```

Running the unit tests shows that we are all "green" at this point. Excellent,  that means we're done, right? The **Truncate** method does what we  expect it to?

Well, yes, it does what we expect it do in this particular case, but something  doesn't feel right about the current implementation. It seems too simple ;-)

Specifically, the code doesn't look like it will handle the scenario where the  input string doesn't contain any spaces.

Let's add another test case to see what happens in this scenario:

```
/// <summary>
        /// Validates that an input string without a space is truncated as
        /// expected.
        /// </summary>
        [TestMethod()]
        public void Truncate004()
        {
            const string input = "foobar";
            const int maxLength = 5;
            const string expected = "fo...";

            string actual = StringHelper.Truncate(input, maxLength);

            Assert.AreEqual(expected, actual);
        }
```

Sure enough, this unit test fails -- and not in a good way (meaning our call  to `Assert.AreEqual` fails, but rather  we aren't even getting that far due to an **ArgumentOutOfRangeException**).

Let's add a check inside the **Truncate** method to handle the scenario  where we fail to find a space in the input string:

```
public static string Truncate(
            string input,
            int maxLength)
        {
            if (maxLength < 4)
            {
                throw new ArgumentException(
                    "A string cannot be truncated to less than 4 characters.",
                    "maxLength");
            }

            if (string.IsNullOrEmpty(input))
            {
                return input;
            }

            int index = input.LastIndexOf(' ', maxLength - 3);

            if (index == -1)
            {
                index = maxLength - 3;
            }

            string truncatedText = input.Substring(0, index) + "...";

            return truncatedText;
        }
```

Now all of the unit tests pass, so we're done, right?

Hold on a minute...

Let's test the boundaries a little by adding yet another unit test:

```
/// <summary>
        /// Validates that the input string is not truncated when maxLength is
        /// sufficiently large.
        /// </summary>
        [TestMethod()]
        public void Truncate005()
        {
            const string input = "The quick brown fox jumped over the lazy dog.";
            int maxLength = input.Length;
            const string expected = input;

            string actual = StringHelper.Truncate(input, maxLength);

            Assert.AreEqual(expected, actual);
        }
```

Sure enough, this new test fails, because the string is truncated and the ellipsis  added -- even though the maximum length specified is sufficient to hold the entire  input string.

Consequently, we need to add a little more code to the **Truncate** method:

```
public static string Truncate(
            string input,
            int maxLength)
        {
            if (maxLength < 4)
            {
                throw new ArgumentException(
                    "A string cannot be truncated to less than 4 characters.",
                    "maxLength");
            }

            if (string.IsNullOrEmpty(input))
            {
                return input;
            }
            else if (input.Length <= maxLength)
            {
                return input;
            }

            int index = input.LastIndexOf(' ', maxLength - 3);

            if (index == -1)
            {
                index = maxLength - 3;
            }

            string truncatedText = input.Substring(0, index) + "...";

            return truncatedText;
        }
```

At this point, we could very well call the **Truncate** method done  and move on to more challenging tasks.

However, suppose that instead of using the **LastIndexOf** method  to find a space in the input string, we mistakenly used the **IndexOf** method to search for a space from the beginning of the string. What would  happen?

Well, unfortunately, all of the unit tests created so far would still pass. That's  certainly not good.

> **Important**
>
> While you might be tempted to modify the **Truncate003** unit test to specify a string with multiple spaces in it, my recommendation is you resist the temptation to modify existing unit tests.

The reason for this is that when familiarizing myself with code that I didn't  write, I find it very helpful to be able to read through unit tests almost like  a spec, meaning that you typically start at a high level (or the simplest scenario)  and then gradually work your way down through more and more complex scenarios.

Consequently, let's add one last test case for the **StringHelper.Truncate** method:

```
/// <summary>
        /// Validates that the input string is truncated as expected.
        /// </summary>
        [TestMethod()]
        public void Truncate006()
        {
            const string input = "The quick brown fox jumped over the lazy dog.";
            const int maxLength = 29;
            const string expected = "The quick brown fox jumped...";

            string actual = StringHelper.Truncate(input, maxLength);

            Assert.AreEqual(expected, actual);
        }
```

At this point, it seems we can finally consider the feature "done."

Hopefully this simple example shows you how TDD causes you to think differently  about writing code -- at least that's one of the most profound effects it has had  on me. In particular, I find that it helps me avoid bugs later on by focusing on  ensuring the code I write handles the simplest "happy path" scenario as well as  other permutations and boundary cases.

Just remember that there will almost always be bugs in the code you (or I) write  -- but whether or not these bugs manifest themselves and negatively impact your  solution is a different matter.

In my [next post](/blog/jjameson/2010/04/15/test-driven-development-tdd-in-the-real-world-part-3-a-k-a-the-encryptionservice), I'll discuss an example of using TDD for a scenario that is considerably  more complex than simply truncating a string.

