---
title: "Test-Driven Development (TDD) in the Real World, Part 3 (a.k.a. the EncryptionService)"
date: 2010-04-15T10:22:00-06:00
excerpt: "In my previous post , I provided a walkthrough of Test-Driven Development (TDD), based on a very simple scenario (truncating a string to a specific number of characters). In this post, I'll provide another example using a more complex scenario. 
 Suppose..."
aliases: ["/blog/jjameson/archive/2010/04/14/test-driven-development-tdd-in-the-real-world-part-3-a-k-a-the-encryptionservice.aspx", "/blog/jjameson/archive/2010/04/15/test-driven-development-tdd-in-the-real-world-part-3-a-k-a-the-encryptionservice.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/04/15/test-driven-development-tdd-in-the-real-world-part-3-a-k-a-the-encryptionservice.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/04/15/test-driven-development-tdd-in-the-real-world-part-3-a-k-a-the-encryptionservice.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

In my [previous post](/blog/jjameson/2010/04/09/test-driven-development-tdd-in-the-real-world-part-2), I provided a walkthrough of Test-Driven Development (TDD), based on a very simple scenario (truncating a string to a specific number of characters). In this post, I'll provide another example using a more complex scenario.

Suppose that we are developing a Web application and we need to encrypt some sensitive data, such as credentials used to access an external system. In other words, we need our Web application to support single-sign on (SSO) in order to access or display data from another system, and we want to store the SSO credentials in encrypted form. [Let's assume we're not going to leverage the Transparent Data Encryption (TDE) features in SQL Server for this particular scenario, which could potentially eliminate the need for us to encrypt and decrypt the data ourselves.]

Each user of the Web application will specify his or her username/password for accessing the external system, which we will subsequently encrypt and store in the user's profile. Consequently, we want to create a class with an **Encrypt** method that encapsulates the details (such as which encryption method to use).

The .NET Framework makes it relatively easy to encrypt and decrypt data using a variety of different algorithms, so it shouldn't take much work to create the **Encrypt** method (and corresponding **Decrypt** method).

However, in my experience, the most difficult part of encrypting data is managing the keys used to encrypt and decrypt the data. For this scenario, let's assume the key used to encrypt the data is managed internally by the encryption service. In other words, each user's SSO credentials are encrypted using the same key.

Let's start by writing a couple of unit tests. However, before we do that we first need to decide where to put the unit tests. Start by creating a new C# **Class Library** project called **Security**, then create a corresponding project using the C# **Test Project** template called **Security.DeveloperTests**.

At this point, I also recommend changing the default namespaces and assembly names to something more meaningful, like **Fabrikam.Demo.Security**, as well as configuring several other options such as enabling code analysis and treating all warnings as errors (including both compilation warnings as well as code analysis warnings). I also recommend [configuring shared assembly information](/blog/jjameson/2009/04/03/shared-assembly-info-in-visual-studio-projects) and signing the assemblies with a strong name key.

The second thing we need to do is decide on a class name for the **Encrypt** and **Decrypt** methods. How about **EncryptionService**?

Now we can write a couple of very simple unit tests:

```
using System;

using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace Fabrikam.Demo.Security.DeveloperTests
{
    /// <summary>
    /// Contains unit tests for the
    /// <see cref="Fabrikam.Demo.Security.EncryptionService" />
    /// class.
    /// </summary>
    [TestClass]
    public class EncryptionServiceTest
    {
        private TestContext testContextInstance;

        /// <summary>
        ///Gets or sets the test context which provides
        ///information about and functionality for the current test run.
        ///</summary>
        public TestContext TestContext
        {
            get
            {
                return testContextInstance;
            }
            set
            {
                testContextInstance = value;
            }
        }

        /// <summary>
        /// Validates that a simple string is encrypted successfully.
        /// </summary>
        [TestMethod]
        public void Encrypt001()
        {
            string plaintext = "foobar";

            string ciphertext = EncryptionService.Encrypt(plaintext);

            Assert.IsFalse(string.IsNullOrEmpty(ciphertext));
            Assert.AreNotEqual<string>(plaintext, ciphertext);
        }

        /// <summary>
        /// Validates that a simple string is encrypted and subsequently
        /// decrypted successfully.
        /// </summary>
        [TestMethod]
        public void Decrypt001()
        {
            string plaintext = "foobar";
            string expected = plaintext;
            string ciphertext = EncryptionService.Encrypt(plaintext);

            string actual = EncryptionService.Decrypt(ciphertext);

            Assert.AreEqual<string>(expected, actual);
        }
    }
}
```

As you can see, my definition of encrypting a simple string "successfully" is very basic. Actually, you might call it laughable, since all I'm doing is ensuring we don't get back a null or empty string, as well as verifying the encrypted text (i.e. *ciphertext*) is not the same as the original text (i.e. *plaintext*). However, I'm assuming that the unit test for the **Decrypt** method will actually verify the **EncryptionService** is doing what we need it to do. One could argue that the **Encrypt001** unit test doesn't really add any value and therefore could be eliminated, but let's keep it for the sake of clarity.

If we attempt to build the solution at this point, we get some compilation errors because the **EncryptionService** doesn't actually exist. Consequently, let's add the corresponding "shell" to the **Security** project:

```
namespace Fabrikam.Demo.Security
{
    /// <summary>
    /// Provides basic services for encrypting sensitive data.
    /// </summary>
    /// <remarks>
    /// All methods of the <c>EncryptionService</c> class are static and can
    /// therefore be called without creating an instance of the class.
    /// </remarks>
    public static class EncryptionService
    {
        /// <summary>
        /// Encrypts the specified text.
        /// </summary>
        public static string Encrypt(
            string plaintext)
        {
            return plaintext;
        }

        /// <summary>
        /// Decrypts the specified text.
        /// </summary>
        public static string Decrypt(
            string ciphertext)
        {
            return ciphertext;
        }
    }
}
```

Note that at this point, we're not worried about the actual implementation of the **Encrypt** and **Decrypt** methods. Rather, we just want to get the solution to compile so that we can ensure our new unit tests are "red" (i.e. they fail) before we actually start working on making them "green" (i.e. pass).

However, we have a problem...

The **Decrypt001** unit test actually passes -- even though we aren't really encrypting and decrypting the specified text!

Well, that certainly isn't good. Let's modify the unit test a little to ensure that it fails based on the current implementation:

```
        [TestMethod]
        public void Decrypt001()
        {
            string plaintext = "foobar";
            string expected = plaintext;
            string ciphertext = EncryptionService.Encrypt(plaintext);

            string actual = EncryptionService.Decrypt(ciphertext);

            Assert.AreEqual<string>(expected, actual);
            Assert.AreNotEqual<string>(plaintext, ciphertext);
        }
```

That's better...now both of our unit tests fail due to the following error:

{{< blockquote "font-italic text-danger" >}}

Assert.AreNotEqual failed. Expected any value except:&lt;foobar&gt;. Actual:&lt;foobar&gt;.

{{< /blockquote >}}

Perhaps you'd rather see a more meaningful message when the test fails. In that case, you can specify the optional `message` parameter when using one of the methods on the **Assert** class:

```
            Assert.AreNotEqual<string>(
                plaintext,
                ciphertext,
                "The encrypted text (ciphertext) should not be the same as the"
                    + " unencrypted text (plaintext).");
```

With this change, the unit tests would fail with the following error:

{{< blockquote "font-italic text-danger" >}}

Assert.AreNotEqual failed. Expected any value except:&lt;foobar&gt;. Actual:&lt;foobar&gt;. The encrypted text (ciphertext) should not be the same as the unencrypted text (plaintext).

{{< /blockquote >}}

Personally, I typically don't see the value in specifying this additional parameter most of the time because it should be obvious why the test failed once you examine the unit test. Although not true in this particular case, there are times when adding your own failure message helps to clarify what is being validated by the unit test.

Now let's focus on getting our unit tests to pass.

As I mentioned before, the most difficult part of encrypting data is managing the keys used to encrypt and decrypt the data. For this scenario, I mentioned that the goal is to provide a mechanism for securely storing SSO credentials for a Web application.

If you are familiar with ASP.NET, you are probably aware that you can configure the [**SqlMembershipProvider**](http://msdn.microsoft.com/en-us/library/system.web.security.sqlmembershipprovider.aspx) to store passwords either in clear, encrypted, or hashed form. Here is some corresponding text from the MSDN page for **[SqlMembershipProvider.PasswordFormat](http://msdn.microsoft.com/en-us/library/system.web.security.sqlmembershipprovider.passwordformat.aspx)** property:

{{< blockquote "font-italic" >}}

Encrypted and Hashed passwords are encrypted or hashed by default based on information supplied in the [machineKey](http://msdn.microsoft.com/en-us/library/w8h3skw9.aspx) element in your configuration.

{{< /blockquote >}}

If other words, if you encrypt passwords using the **SqlMembershipProvider**, it uses a symmetric-key algorithm based on the machineKey element in Web.config. Assuming you implement the necessary security around your Web.config files, this mitigates the difficulty in managing the key necessary to encrypt and decrypt data.

The actual implementation for encrypting a password is provided by the **[EncryptPassword](http://msdn.microsoft.com/en-us/library/ms152042.aspx)** method. Similarly, decrypting a password is provided by the **[DecryptPassword](http://msdn.microsoft.com/en-us/library/system.web.security.membershipprovider.decryptpassword.aspx)** method. Thus with very little effort, we can implement the necessary functionality to encrypt/decrypt arbitrary text (e.g. SSO credentials).

Note, however, that the **EncryptPassword** and **DecryptPassword** methods are `protected` (not `public`). Consequently, without reverting to some unsupported or poorly performing hack (e.g. using reflection to call the protected methods), we need to inherit from the **SqlMembershipProvider** class in order to use this functionality.

Having our custom **EncryptionService** class inherit directly from **SqlMembershipProvider** seems like a bad idea (since our goal is simply to provide encryption services -- not a full-blown membership provider). Therefore, let's create a new class called **InternalEncryptionService** that, unlike **EncryptionService**, is scoped as `internal`.

In other words, the **EncryptionService** class will delegate the work of actually encrypting/decrypting the data to the **InternalEncryptionService** class, which in turn delegates the work to the out-of-the-box functionality provided by ASP.NET (by inheriting from **SqlMembershipProvider**).

While we could certainly rely on the unit tests already developed to (indirectly) test the new **InternalEncryptionService** class, I prefer to create unit tests at the "lower layers" so that in the event something breaks, I can start investigating from the lowest layer (thus minimizing the amount of code I need to debug).

With that in mind, let's copy/paste the unit tests for the **EncryptionService** class (i.e. EncryptionServiceTest.cs) to create unit tests for the **InternalEncryptionService** class:

```
using System;

using Microsoft.VisualStudio.TestTools.UnitTesting;

using Fabrikam.Demo.Security;

namespace Fabrikam.Demo.Security.DeveloperTests
{
    /// <summary>
    /// Contains unit tests for the
    /// <see cref="Fabrikam.Demo.Security.InternalEncryptionService" />
    /// class.
    /// </summary>
    [TestClass]
    public class InternalEncryptionServiceTest
    {
        private TestContext testContextInstance;

        /// <summary>
        ///Gets or sets the test context which provides
        ///information about and functionality for the current test run.
        ///</summary>
        public TestContext TestContext
        {
            get
            {
                return testContextInstance;
            }
            set
            {
                testContextInstance = value;
            }
        }

        /// <summary>
        /// Validates that a simple string is encrypted successfully.
        /// </summary>
        [TestMethod]
        public void Encrypt001()
        {
            string plaintext = "foobar";

            string ciphertext = InternalEncryptionService.Encrypt(plaintext);

            Assert.IsFalse(string.IsNullOrEmpty(ciphertext));
            Assert.AreNotEqual<string>(plaintext, ciphertext);
        }

        /// <summary>
        /// Validates that a simple string is encrypted and subsequently
        /// decrypted successfully.
        /// </summary>
        [TestMethod]
        public void Decrypt001()
        {
            string plaintext = "foobar";
            string expected = plaintext;
            string ciphertext = InternalEncryptionService.Encrypt(plaintext);

            string actual = InternalEncryptionService.Decrypt(ciphertext);

            Assert.AreEqual<string>(expected, actual);
            Assert.AreNotEqual<string>(plaintext, ciphertext);
        }
    }
}
```

> **Note**
>
> When doing TDD, we typically want to work in very small increments (i.e. get our existing tests to pass before adding more complexity). However, in this case, it makes sense to add a couple more failing unit tests (as well as another class) because our goal is to implement the **Encrypt** and **Decrypt** methods with as little work (i.e. custom code) as possible.

Next, copy/paste the **EncryptionService** class (i.e. EncryptionService.cs) to create the **InternalEncryptionService** class and make the necessary changes to inherit from **SqlMembershipProvider** (note that you'll need to add references to System.Configuration and System.Web):

```
using System.Web.Security;

namespace Fabrikam.Demo.Security
{
    /// <summary>
    /// Provides basic services for encrypting sensitive data.
    /// </summary>
    /// <remarks>
    /// This class derives from SqlMembershipProvider in order to leverage the
    /// EncryptPassword and DecryptPassword methods.
    /// </remarks>
    internal class InternalEncryptionService : SqlMembershipProvider
    {
        /// <summary>
        /// Encrypts the specified text.
        /// </summary>
        public static string Encrypt(
            string plaintext)
        {
            return plaintext;
        }

        /// <summary>
        /// Decrypts the specified text.
        /// </summary>
        public static string Decrypt(
            string ciphertext)
        {
            return ciphertext;
        }
    }
}
```

Attempting to build the solution at this point results in the following error:

{{< blockquote "font-italic text-danger" >}}

'Fabrikam.Demo.Security.InternalEncryptionService' is inaccessible due to its protection level

{{< /blockquote >}}

This makes sense because **InternalEncryptionService** is marked as internal, yet we are trying to access it from the separate **Fabrikam.Demo.Security.DeveloperTests** assembly.

Note that if you create your unit tests using some built-in features of Visual Studio, it will automatically create "wrapper" classes that you can use in unit testing scenarios like this. However, since I tend to just create unit tests via code (often via copy/paste like I explained above), I don't typically rely on these wrapper classes.

An alternative for this scenario -- since our class is marked as `internal` -- is to add an **[InternalsVisibleToAttribute](http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.internalsvisibletoattribute.aspx)** to the AssemblyInfo.cs file for the **Security** project (which creates the **Fabrikam.Demo.Security** assembly):

```
using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

// Note: Shared assembly information is specified in SharedAssemblyInfo.cs

// General Information about an assembly is controlled through the following
// set of attributes. Change these attribute values to modify the information
// associated with an assembly.
[assembly: AssemblyTitle("Fabrikam.Demo.Security")]
[assembly: AssemblyCulture("")]

// The following GUID is for the ID of the typelib if this project is exposed to COM
[assembly: Guid("7ac9b463-c89b-425a-aeae-c6cc8c318aea")]

[assembly: InternalsVisibleTo("Fabrikam.Demo.Security.DeveloperTests, PublicKey="
    + "00240000048000009400000006020000002400005253413100040000010001008748be47"
        + "c45d376f413042b18521c05affcfdfcbf7d73c7273acdf5cd1a056bc4d460dceee16"
        + "92d1f33fa16f8f7f3afd6c75552e8bfaa1ebe6fabf8f7923d48697bba4e22c8fad0e"
        + "0b3e266ff5266292e22254b567f51c80ce404188643aa17a1378eff241ed01a36b3d"
        + "64c127334a0ba4eec58f95f3606e73e103053006d0bf")]
```

To extract the public key from the assembly, use the "-Tp" option on the [Strong Name tool (Sn.exe)](http://msdn.microsoft.com/en-us/library/k5b5tt23%28VS.80%29.aspx), as demonstrated below:

{{< console-block-start >}}

C:\NotBackedUp\Fabrikam\Demo\Main\Source\Security\DeveloperTests\bin\Debug&gt;{{< kbd "sn -Tp Fabrikam.Demo.Security.DeveloperTests.dll" >}}

{{< sample-block >}}

Microsoft (R) .NET Framework Strong Name Utility Version 3.5.30729.1\ Copyright (c) Microsoft Corporation. All rights reserved.\ \ Public key is\ 00240000048000009400000006020000002400005253413100040000010001008748be47c45d37\ 6f413042b18521c05affcfdfcbf7d73c7273acdf5cd1a056bc4d460dceee1692d1f33fa16f8f7f\ 3afd6c75552e8bfaa1ebe6fabf8f7923d48697bba4e22c8fad0e0b3e266ff5266292e22254b567\ f51c80ce404188643aa17a1378eff241ed01a36b3d64c127334a0ba4eec58f95f3606e73e10305\ 3006d0bf\ \ Public key token is 786f58ca4a6e3f60

{{< /sample-block >}}

{{< console-block-end >}}

With the **InternalsVisibleToAttribute** specifed, the solution builds and we now have four failing unit tests (instead of just the two that we had before).

> **Tip**
>
> If you tend to run your unit tests in Visual Studio using the **Test List Editor** (like I do) then I recommend adding the **Full Class Name** column to the view (in order to resolve any ambiguity between unit tests and easily identify the class where the unit test is implemented). You should also consider adding this column to the **Test Results** window. [Personally, I find the **Full Class Name** column to be much more valuable than the **Project** column that gets added by default.]

Now let's focus on getting the two unit tests for the **InternalEncryptionService** class to pass.

Start by replacing the implementation of the **Encrypt** method in the **InternalEncryptionService** class:

```
        public string Encrypt(
            string plaintext)
        {
            byte[] data = Encoding.Unicode.GetBytes(plaintext);

            byte[] encryptedData = base.EncryptPassword(data);

            string ciphertext = Convert.ToBase64String(encryptedData);
            return ciphertext;
        }
```

Similarly, replace the implementation of the **Decrypt** method in the **InternalEncryptionService** class:

```
        public string Decrypt(
            string ciphertext)
        {
            byte[] encryptedData = Convert.FromBase64String(ciphertext);

            byte[] decryptedData = base.DecryptPassword(encryptedData);

            string plainText = Encoding.Unicode.GetString(decryptedData);
            return plainText;
        }
```

Building the solution and running the unit tests now results in a **ProviderException**:

{{< blockquote "font-italic text-danger" >}}

You must specify a non-autogenerated machine key to store passwords in the encrypted format. Either specify a different passwordFormat, or change the machineKey configuration to use a non-autogenerated decryption key.

{{< /blockquote >}}

I don't know about you, but I love error messages like this. It not only provides detailed information on the fundamention problem, it also gives us a "hint" on how to fix it.

Add the following app.config file to the **Security.DeveloperTests** project (to mimic the necessary configuration elements we would normally specify in the Web.config file):

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <system.web>
    <machineKey decryptionKey="18E2D4EF487E48CD69AE82ADEB1495D40B4E437C1A4D231A" />
  </system.web>
</configuration>
```

Now build the solution and run the unit tests.

Voila! The two unit tests for the **InternalEncryptionService** class are now "green" (i.e. passing). Woohoo!

The next step is to get the remaining unit tests (i.e. the two for the **EncryptionService** class) to pass. As noted before, this is simply a matter of delegating the work performed by the **EncryptionService** class to the corresponding methods in the **InternalEncryptionService** class:

```
    public static class EncryptionService
    {
        /// <summary>
        /// Encrypts the specified text.
        /// </summary>
        public static string Encrypt(
            string plaintext)
        {
            InternalEncryptionService service = new InternalEncryptionService();

            return service.Encrypt(plaintext);
        }

        /// <summary>
        /// Decrypts the specified text.
        /// </summary>
        public static string Decrypt(
            string ciphertext)
        {
            InternalEncryptionService service = new InternalEncryptionService();

            return service.Decrypt(ciphertext);
        }
    }
```

Building the solution and running all of the unit tests again confirms that all of our tests are now passing. Woohoo, indeed!

This seems like a good point to check-in the code. While there might be occasions where you check-in code with unit tests that don't pass, in general it's best to only check-in after getting the tests to pass. This is especially true if your new unit tests are considered BVTs (Build Verification Tests), in which case a failing unit test denotes a broken build.

However, even though you check-in the code, does that mean we're done? Heck no!

As I mentioned in my previous post, whenever you write a new piece of code, you should always try to think of ways to "break" it (in order to uncover potential bugs as early as possible).

In the code examples provided so far, did you notice how little error handling is implemented? What happens if we try to encrypt a null or empty string? What should we expect to happen?

We should definitely make the code more robust by validating the parameters in the **InternalEncryptionService** class -- but if we're doing TDD, then we should try to remember to write the unit test first (ensuring that it fails before implementing the necessary work to make it pass). For example, add the following unit test to InternalEncryptionServiceTest.cs:

```
        /// <summary>
        /// Validates that an exception is thrown when the input string is null.
        /// </summary>
        [TestMethod()]
        [ExpectedException(typeof(ArgumentNullException))]
        public void EncryptWithInvalidParameter001()
        {
            const string plaintext = null;

            InternalEncryptionService service = new InternalEncryptionService();

            const string expectedExceptionMessage =
                "Value cannot be null."
                + "\r\nParameter name: plaintext";

            try
            {
                // Prevent code analysis warning (CA1804) by not assigning
                // the return value to a local variable.
                service.Encrypt(plaintext);
            }
            catch (ArgumentNullException ex)
            {
                Assert.AreEqual(expectedExceptionMessage, ex.Message);
                throw;
            }
        }
```

If you run this new unit test, you will find that it fails because, even though an **ArgumentNullException** is thrown, the message specified in the exception ("String reference not set to an instance of a String.\r\nParameter name: s") does not match the expected message.

Add another unit test to validate that an empty string is also handled as expected (since it shouldn't take more than 30 seconds or so to copy/paste and make the necessary changes):

```
        /// <summary>
        /// Validates that an exception is thrown when the input string is empty.
        /// </summary>
        [TestMethod()]
        [ExpectedException(typeof(ArgumentException))]
        public void EncryptWithInvalidParameter002()
        {
            string plaintext = string.Empty;

            InternalEncryptionService service = new InternalEncryptionService();

            const string expectedExceptionMessage =
                "Value cannot be empty."
                + "\r\nParameter name: plaintext";

            try
            {
                // Prevent code analysis warning (CA1804) by not assigning
                // the return value to a local variable.
                service.Encrypt(plaintext);
            }
            catch (ArgumentException ex)
            {
                Assert.AreEqual(expectedExceptionMessage, ex.Message);
                throw;
            }
        }
```

What's interesting at this point is that while the **EncryptWithInvalidParameter002** test fails, the failure isn't due to an unexpected exception message but rather because an exception was *not* thrown. This highlights an interesting scenario -- and perhaps prompts a question that you might never have asked yourself if you weren't using TDD: Should we be able to encrypt an empty string?

According to the ASP.NET team, the answer is "yes" (based on the implementation of the **SqlMembershipProvider** class). In fact, this unit test makes it really easy to step into the debugger and verify that we do indeed get back "garbled" text when calling the **Encrypt** method with an empty string. [Yet another reason to use TDD -- it makes it really easy to dive into the debugger without going through the effort of spinning up a Web application and browsing to the site.]

Assuming we agree with this expected behavior for encrypting an empty string (which seems reasonable), then it makes sense to replace the **EncryptWithInvalidParameter002** test with a different test:

```
        /// <summary>
        /// Validates that an empty string is encrypted successfully.
        /// </summary>
        [TestMethod]
        public void Encrypt002()
        {
            string plaintext = string.Empty;

            InternalEncryptionService service = new InternalEncryptionService();

            string ciphertext = service.Encrypt(plaintext);

            Assert.IsFalse(string.IsNullOrEmpty(ciphertext));
            Assert.AreNotEqual<string>(plaintext, ciphertext);
        }
```

Even though this test might not appear to add any value (since it passes without making any changes to the code), in my opinion it is valuable because it helps others developers (or even me, at some later point in time) quickly understand the expected behavior of the code simply by reading through the unit tests. It also ensures that we don't accidentally change the behavior (based on the assumption that an empty string should be encrypted successfully).

Next we should add similar unit tests to ensure the **Decrypt** method handles null and empty input values as expected. Finally, make the necessary code changes to get the new unit tests to pass and check-in the updated code.

So at this point, are we done? Well, sort of.

Yes, the **EncryptionService** does what we need it to do, but there's one problem -- and it might be considered a relatively big problem depending on your perspective.

At the start, I mentioned that the goal was to be able to encrypt SSO credentials for a Web application. Suppose that both you and I have the same password (merely by coincidence). Should the **EncryptionService** return the same value when encrypting my password and your password? Most security experts would tell you "no, you should use some kind of 'salt' (or 'entropy' as I like to call it) when encrypting values like this."

Consequently we should consider making the **Encrypt** and **Decrypt** methods more robust by adding another parameter:

```
namespace Fabrikam.Demo.Security
{
    /// <summary>
    /// Provides basic services for encrypting sensitive data.
    /// </summary>
    /// <remarks>
    /// All methods of the <c>EncryptionService</c> class are static and can
    /// therefore be called without creating an instance of the class.
    /// </remarks>
    public static class EncryptionService
    {
        /// <summary>
        /// Encrypts the specified text.
        /// </summary>
        /// <param name="plaintext">The text to encrypt.</param>
        /// <returns>The encrypted text ("ciphertext") based on the specified
        /// input.</returns>
        public static string Encrypt(
            string plaintext)
        {
            return Encrypt(plaintext, null);
        }

        /// <summary>
        /// Encrypts the specified text, using the optional entropy (if
        /// specified) to make the encryption more secure.
        /// </summary>
        /// <param name="plaintext">The text to encrypt.</param>
        /// <param name="entropy">An additional "secret" (e.g. password "salt")
        /// that will need to be specified when subsequently decrypting the
        /// encrypted text.</param>
        /// <returns>The encrypted text ("ciphertext") based on the specified
        /// input.</returns>
        public static string Encrypt(
            string plaintext,
            string entropy)
        {
            InternalEncryptionService service = new InternalEncryptionService();

            return service.Encrypt(plaintext, entropy);
        }

        /// <summary>
        /// Decrypts the specified text.
        /// </summary>
        /// <param name="ciphertext">The text to decrypt.</param>
        /// <returns>The decrypted text ("plaintext") based on the specified
        /// input.</returns>
        public static string Decrypt(
            string ciphertext)
        {
            return Decrypt(ciphertext, null);
        }

        /// <summary>
        /// Decrypts the specified text, using the entropy originally specified
        /// when encrypting the data.
        /// </summary>
        /// <param name="ciphertext">The text to decrypt.</param>
        /// <param name="entropy">The additional "secret" (e.g. password "salt")
        /// previously specified when encrypting the data.</param>
        /// <returns>The decrypted text ("plaintext") based on the specified
        /// input.</returns>
        public static string Decrypt(
            string ciphertext,
            string entropy)
        {
            InternalEncryptionService service = new InternalEncryptionService();

            return service.Decrypt(ciphertext, entropy);
        }
    }
}
```

In order to avoid making a breaking change on a "public interface", I simply added new overloads for the methods that allow an optional `entropy` parameter to be specified. However, before I did that, I should have first created some new unit tests to verify that the encrypted text is different depending on whether or not I specify an additional "secret" when calling the **Encrypt** method ;-)

I should also add some unit tests to verify that data encrypted with some entropy can only be decrypted when the same entropy is specified (or an appropriate exception occurs if the wrong entropy value is specified). Then I would move on to updating the **InternalEncryptionService** to actually use the `entropy` parameter (if it is specified). If an entropy is not specified, then I should ensure the unit tests created before adding the additional parameter still pass as expected.

Note that some changes might be needed to the previously developed unit tests -- depending on how the code is modified. For example, is the `entropy` parameter optional in the methods on the **InternalEncryptionService** class as well? If no, then we'll need to update the old unit tests to pass in `null` for the `entropy` parameter.

> **Important**
>
> Whenever you are changing existing unit tests (for the purposes of refactoring or other reasons), be careful not to mistakenly change the intent of the unit test -- unless, of course, the unit test fails because of an *expected* change in behavior of the underlying code.

Given the length of this post, I'll leave the rest of the work on the encryption service as an exercise for the reader ;-)

