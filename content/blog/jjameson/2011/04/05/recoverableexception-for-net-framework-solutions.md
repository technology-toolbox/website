---
title: "RecoverableException for .NET Framework Solutions"
date: 2011-04-05T05:38:00-06:00
excerpt: "Do you remember the good ol' days before the ApplicationException class in the .NET Framework became \" persona non grata \"? I sure do. 
 If you were to look at .NET code that I wrote years ago, you'd probably see ApplicationException being used all over..."
aliases: ["/blog/jjameson/archive/2011/04/04/recoverableexception-for-net-framework-solutions.aspx", "/blog/jjameson/archive/2011/04/05/recoverableexception-for-net-framework-solutions.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["My System", "Core Development"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2011/04/05/recoverableexception-for-net-framework-solutions.aspx](http://blogs.msdn.com/b/jjameson/archive/2011/04/05/recoverableexception-for-net-framework-solutions.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog ever goes away.

Do you remember the good ol' days before the **ApplicationException** class in the .NET Framework became "[persona non grata](http://en.wikipedia.org/wiki/Persona_non_grata)"? I sure do.

If you were to look at .NET code that I wrote years ago, you'd probably see **ApplicationException** being used all over the place. After all, this seemed like a great way to differentiate between "expected" errors in your code (e.g. a violation of some business rule) vs. truly "unexpected" errors (e.g. an invalid parameter passed to a method).

Since we didn't expect instances of **ApplicationException** to be thrown by the .NET Framework itself, it seemed like a good idea to use **ApplicationException** as a way to determine which errors should be caught and shown to end users (i.e. "handled gracefully") vs. errors for which there's rarely any point in catching. For example, what is a user really going to do when you tell them "Object reference not set to an instance of an object?" Besides, of course, to curse your application under their breath or, even worse, openly vent their frustration to everyone within shouting distance.

Since we shouldn't be using **ApplicationException** anymore, we need some other way of distinguishing different types of errors. Personally, I recommend throwing a custom **RecoverableException** (or an exception that derives from **RecoverableException**) in scenarios where the user -- or an administrator -- can take some action in order to resolve the error. In other words, somebody -- besides a developer -- should be able to understand what the error means and consequently do *something* in order to recover from the error. Note that the "something" might simply be a matter of waiting a short period and trying the operation again (for example, when a remote Web service returns a "server too busy" error).

Here's an example implementation of a custom exception class to support these scenarios:

```
using System;
using System.Runtime.Serialization;
using System.Security.Permissions;

namespace Fabrikam.Demo.CoreServices
{
    /// <summary>
    /// The exception that is thrown when an error occurs that can be recovered
    /// by a user. The exception message is presumed to be safe to display to
    /// users (in other words, the error message does not contain any sensitive
    /// information).
    /// </summary>
    /// <remarks>
    /// A "recoverable" error may be caused by a transient condition (e.g. a
    /// remote Web service is too busy), in which case the request can simply be
    /// resubmitted at a later time; or the "recoverable" error may require user
    /// action in order to resolve the condition that caused the error (e.g.
    /// the SSO credentials for an external system are not specified in the
    /// user's profile).
    /// </remarks>
    [Serializable]
    public class RecoverableException : Exception, ISerializable
    {
        /// <summary>
        /// Initializes a new instance of the <see cref="RecoverableException"/>
        /// class.
        /// </summary>
        public RecoverableException()
        {
        }

        /// <summary>
        /// Initializes a new instance of the <see cref="RecoverableException"/>
        /// class, using the specified message.
        /// </summary>
        /// <param name="message">The description of the error condition.
        /// </param>
        public RecoverableException(
            string message) : base(message)
        {
        }

        /// <summary>
        /// Initializes a new instance of the <see cref="RecoverableException"/>
        /// class, using the specified message and the inner exception.
        /// </summary>
        /// <param name="message">The description of the error condition.
        /// </param>
        /// <param name="innerException">The inner exception to be used.</param>
        public RecoverableException(
            string message,
            Exception innerException) : base(message, innerException)
        {
        }

        /// <summary>
        /// Initializes a new instance of the <see cref="RecoverableException"/>
        /// class, using the specified serialization information and context
        /// objects.
        /// </summary>
        /// <param name="info">Information relevant to the deserialization
        /// process.</param>
        /// <param name="context">The context of the deserialization
        /// process.</param>
        protected RecoverableException(
            SerializationInfo info,
            StreamingContext context) : base(info, context)
        {
        }

        #region ISerializable Members

        /// <summary>
        /// Sets the
        /// <see cref="System.Runtime.Serialization.SerializationInfo"/>
        /// with information about the exception.
        /// </summary>
        /// <param name="info">The
        /// <see cref="System.Runtime.Serialization.SerializationInfo"/> that
        /// holds the serialized object data about the exception being thrown.
        /// </param>
        /// <param name="context">The
        /// <see cref="System.Runtime.Serialization.StreamingContext"/> that
        /// contains contextual information about the source or destination.
        /// </param>
        [SecurityPermission(
            SecurityAction.LinkDemand,
            Flags = SecurityPermissionFlag.SerializationFormatter)]
        public override void GetObjectData(
            SerializationInfo info,
            StreamingContext context)
        {
            base.GetObjectData(info, context);
        }

        #endregion
    }
}
```

The following code snippet shows an example of catching "expected" and "unexpected" exceptions and handling the scenarios accordingly:

```
            try
            {
                BindSiteList();

                SelectDefaultSiteForExport();
            }
            catch (RecoverableException ex)
            {
                SiteList.Items.Clear();

                DelayedLoadErrorMessage.Text = ex.Message;
            }
            catch (Exception ex)
            {
                Logger.LogError(ex);

                SiteList.Items.Clear();

                DelayedLoadErrorMessage.Text =
                    Resources.Error_UnexpectedError_ReferToEventLog);
            }
```

Note that it is generally a bad idea to catch instances of the **Exception** class. In this particular instance the code snippet is from a "delayed load" event for an ASP.NET **UpdatePanel**. In order to populate the list of sites on the page (in other words, "bind the site list") a call is made to a remote application using single sign-on (SSO) credentials configured for the current user. In order to gracefully handle errors during the AJAX partial page update, I catch instances of **RecoverableException** for error conditions that can be resolved by a user (for example, when the SSO credentials have not been configured for the user) and **Exception** for other errors (for example, when attempting to retrieve the list of sites from the remote application generates an unexpected error).

> **Note**
>
> A more elegant way of handling errors during asynchronous postbacks is to use the **[ScriptManager.AsyncPostBackError](http://msdn.microsoft.com/en-us/library/system.web.ui.scriptmanager.asyncpostbackerror.aspx)** event. I'll cover that in a separate post.

