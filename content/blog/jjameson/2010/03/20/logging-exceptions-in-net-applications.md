---
title: "Logging Exceptions in .NET Applications"
date: 2010-03-20T05:08:00-06:00
excerpt: "Last summer I wrote a post introducing my simple, but highly effective approach to logging -- including a Logger class that is really just a thin wrapper around the System.Diagnostics.TraceSource class. 
 A few months ago, I enhanced the Logger class..."
aliases: ["/blog/jjameson/archive/2010/03/19/logging-exceptions-in-net-applications.aspx", "/blog/jjameson/archive/2010/03/20/logging-exceptions-in-net-applications.aspx"]
draft: true
categories: ["My System", "SharePoint", "Development"]
tags: ["My System", "Simplify", "
                    MOSS 2007", "Core
                        Development", "WSS v3"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/03/20/logging-exceptions-in-net-applications.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/03/20/logging-exceptions-in-net-applications.aspx)
>
> Since [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog                 ever goes away.

Last summer I wrote a post introducing [my simple, but highly effective approach to logging](/blog/jjameson/2009/06/18/a-simple-but-highly-effective-approach-to-logging) -- including a **Logger** class that is really just a thin wrapper around the [System.Diagnostics.TraceSource](http://msdn.microsoft.com/en-us/library/system.diagnostics.tracesource%28VS.80%29.aspx) class.

A few months ago, I enhanced the **Logger** class to log exceptions         in a consistent fashion.

I used the "[Yellow
Page of Death](http://en.wikipedia.org/wiki/Yellow_Screen_of_Death#Yellow)" provided by ASP.NET as a reference for logging the details         of the exception. Here's a screenshot from a sample that I whipped up this morning:

{{< figure
src="https://assets.technologytoolbox.com/blog/jjameson/Images/Development/ASP.NET-Yellow-Page-of-Death-600x295.png"
alt="ASP.NET error page"
class="screenshot"
height="295"
width="600"
title="Figure 1: ASP.NET error page" >}}

[See full-sized image.](https://assets.technologytoolbox.com/blog/jjameson/Images/Development/ASP.NET-Yellow-Page-of-Death-1301x639.png)

The corresponding code-behind from the page that I used to generate the above screenshot         is listed below:

```
using System;

using Fabrikam.Demo.CoreServices.Logging;

namespace Fabrikam.Demo.Web.UI.Logging
{
    public partial class UnhandledExceptionPage : System.Web.UI.Page
    {
        protected void Page_Load(
            object sender,
            EventArgs e)
        {
            DoSomethingBad();
        }

        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Performance",
            "CA1804:RemoveUnusedLocals",
            MessageId = "divideByZero")]
        private static void DoSomethingBad()
        {
            int numerator = 1;
            int denominator = 0;

            try
            {
                int divideByZero = numerator / denominator;
            }
            catch (DivideByZeroException ex)
            {
                string errorMessage = "Something bad happened.";
                Logger.LogError(errorMessage);

                throw new InvalidOperationException(
                    errorMessage,
                    ex);
            }
        }
    }
}
```

A couple of interesting notes about the ASP.NET error page:

- The error message displayed at the top of the page is actually from the innermost
  exception (i.e. the "base" exception).
- The stack trace shows the details for the actual exception that occurred as well
  as any inner exceptions.

This makes perfect sense when you think about it because when you start investigating         an error, it's best to begin with the crux of the problem. In other words, for this         example, it's much better start off with the             "Attempted to divide by zero" message from the **DivideByZeroException**,         rather than the "Something bad happened"         message from the **InvalidOperationExeption**.

In order to log exceptions, I added two new overloads for the **LogError**         method:

```
/// <summary>
        /// Logs the specified exception to the trace listeners.
        /// </summary>
        /// <param name="ex">The exception to log.</param>
        public static void LogError(
            Exception ex)
        {
            LogError(ex, null);
        }

        /// <summary>
        /// Logs the specified exception to the trace listeners.
        /// </summary>
        /// <remarks>The details of the exception are logged using a format
        /// similar to the ASP.NET error page. Information about the base
        /// exception is logged first, followed by information for any
        /// "outer" exceptions.</remarks>
        /// <param name="ex">The exception to log.</param>
        /// <param name="requestUrl">The URL of the web request for which the
        /// exception occurred.</param>
        public static void LogError(
            Exception ex,
            Uri requestUrl)
        {
            if (ex == null)
            {
                throw new ArgumentNullException("ex");
            }

            StringBuilder buffer = new StringBuilder();

            if (requestUrl == null)
            {
                buffer.Append(
                    "An error occurred in the application.");
            }
            else
            {
                buffer.Append(
                    "An error occurred during the execution of the"
                    + " web request.");
            }

            buffer.Append(
                " Please review the stack trace for more information about the"
                + " error and where it originated in the code.");

            buffer.Append(Environment.NewLine);
            buffer.Append(Environment.NewLine);

            if (requestUrl != null)
            {
                buffer.Append("Request URL: ");
                buffer.Append(requestUrl.AbsoluteUri);

                buffer.Append(Environment.NewLine);
                buffer.Append(Environment.NewLine);
            }

            Exception baseException = ex.GetBaseException();
            Type baseExceptionType = baseException.GetType();

            buffer.Append("Exception Details: ");
            buffer.Append(baseExceptionType.FullName);
            buffer.Append(": ");
            buffer.Append(baseException.Message);

            buffer.Append(Environment.NewLine);
            buffer.Append(Environment.NewLine);

            buffer.Append("Stack Trace:");
            buffer.Append(Environment.NewLine);
            buffer.Append(Environment.NewLine);

            AppendExceptionDetail(ex, buffer);

            string message = buffer.ToString();
            Log(TraceEventType.Error, message);
        }
```

Note that the stack trace is generated using the **AppendExceptionDetail**         helper method.

This makes it really easy to log exceptions in ASP.NET Web applications -- including         solutions built on Microsoft Office SharePoint Server (MOSS) 2007 and Windows SharePoint         Services -- as well as other types of .NET applications (e.g. console applications).

Here is the complete source for the updated **Logger** class:

```
#define TRACE

using System;
using System.Diagnostics;
using System.Globalization;
using System.Text;

namespace Fabrikam.Demo.CoreServices.Logging
{
    /// <summary>
    /// Exposes static methods for logging various events (e.g. debug,
    /// informational, warning, etc.). This class cannot be inherited.
    /// </summary>
    /// <remarks>
    /// All methods of the <b>Logger</b> class are static
    /// and can therefore be called without creating an instance of the class.
    /// </remarks>
    public static class Logger
    {
        private static TraceSource defaultTraceSource =
            new TraceSource("defaultTraceSource");

        private static void AppendExceptionDetail(
            Exception ex,
            StringBuilder buffer)
        {
            Debug.Assert(ex != null);
            Debug.Assert(buffer != null);

            if (ex.InnerException != null)
            {
                AppendExceptionDetail(ex.InnerException, buffer);
            }

            Type exceptionType = ex.GetType();

            buffer.Append('[');
            buffer.Append(exceptionType.Name);
            buffer.Append(": ");
            buffer.Append(ex.Message);
            buffer.Append(']');
            buffer.Append(Environment.NewLine);

            // Note: Exception.StackTrace includes both _stackTrace and
            // _remoteStackTrace, so use System.Diagnostics.StackTrace
            // to retrieve only the portion we want to output
            StackTrace trace = new StackTrace(ex);
            buffer.Append(trace.ToString());

            buffer.Append(Environment.NewLine);
        }

        /// <summary>
        /// Flushes all the trace listeners in the trace listener collection.
        /// </summary>
        public static void Flush()
        {
            defaultTraceSource.Flush();
        }

        /// <summary>
        /// Logs an event to the trace listeners using the specified
        /// event type and message.
        /// </summary>
        /// <param name="eventType">One of the System.Diagnostics.TraceEventType
        /// values that specifies the type of event being logged.</param>
        /// <param name="message">The message to log.</param>
        public static void Log(
            TraceEventType eventType,
            string message)
        {
#if DEBUG
            // Some debug listeners (e.g. DbgView.exe) don't buffer output, so
            // Debug.Write() is effectively the same as Debug.WriteLine().
            // For optimal appearance in these listeners, format the output
            // for a single call to Debug.WriteLine().
            StringBuilder sb = new StringBuilder();

            sb.Append(eventType.ToString());
            sb.Append(": ");
            sb.Append(message);

            string formattedMessage = sb.ToString();
            Debug.WriteLine(formattedMessage);
#endif

            defaultTraceSource.TraceEvent(eventType, 0, message);
        }

        /// <summary>
        /// Logs a debug event to the trace listeners using the specified
        /// format string and arguments.
        /// </summary>
        /// <param name="provider">An System.IFormatProvider that supplies
        /// culture-specific formatting information.</param>
        /// <param name="format">A composite format string.</param>
        /// <param name="args">An System.Object array containing zero or more
        /// objects to format.</param>
        public static void LogDebug(
            IFormatProvider provider,
            string format,
            params object[] args)
        {
            string message = string.Format(
                provider, format, args);

            LogDebug(message);
        }

        /// <summary>
        /// Logs a debug event to the trace listeners.
        /// </summary>
        /// <param name="message">The message to log.</param>
        public static void LogDebug(
            string message)
        {
            Log(TraceEventType.Verbose, message);
        }

        /// <summary>
        /// Logs a critical event to the trace listeners using the specified
        /// format string and arguments.
        /// </summary>
        /// <param name="provider">An System.IFormatProvider that supplies
        /// culture-specific formatting information.</param>
        /// <param name="format">A composite format string.</param>
        /// <param name="args">An System.Object array containing zero or more
        /// objects to format.</param>
        public static void LogCritical(
            IFormatProvider provider,
            string format,
            params object[] args)
        {
            string message = string.Format(
                provider, format, args);

            LogCritical(message);
        }

        /// <summary>
        /// Logs a critical event to the trace listeners.
        /// </summary>
        /// <param name="message">The message to log.</param>
        public static void LogCritical(
            string message)
        {
            Log(TraceEventType.Critical, message);
        }

        /// <summary>
        /// Logs an error event to the trace listeners using the specified
        /// format string and arguments.
        /// </summary>
        /// <param name="provider">An System.IFormatProvider that supplies
        /// culture-specific formatting information.</param>
        /// <param name="format">A composite format string.</param>
        /// <param name="args">An System.Object array containing zero or more
        /// objects to format.</param>
        public static void LogError(
            IFormatProvider provider,
            string format,
            params object[] args)
        {
            string message = string.Format(
                provider, format, args);

            LogError(message);
        }

        /// <summary>
        /// Logs the specified exception to the trace listeners.
        /// </summary>
        /// <param name="ex">The exception to log.</param>
        public static void LogError(
            Exception ex)
        {
            LogError(ex, null);
        }

        /// <summary>
        /// Logs the specified exception to the trace listeners.
        /// </summary>
        /// <remarks>The details of the exception are logged using a format
        /// similar to the ASP.NET error page. Information about the base
        /// exception is logged first, followed by information for any
        /// "outer" exceptions.</remarks>
        /// <param name="ex">The exception to log.</param>
        /// <param name="requestUrl">The URL of the web request for which the
        /// exception occurred.</param>
        public static void LogError(
            Exception ex,
            Uri requestUrl)
        {
            if (ex == null)
            {
                throw new ArgumentNullException("ex");
            }

            StringBuilder buffer = new StringBuilder();

            if (requestUrl == null)
            {
                buffer.Append(
                    "An error occurred in the application.");
            }
            else
            {
                buffer.Append(
                    "An error occurred during the execution of the"
                    + " web request.");
            }

            buffer.Append(
                " Please review the stack trace for more information about the"
                + " error and where it originated in the code.");

            buffer.Append(Environment.NewLine);
            buffer.Append(Environment.NewLine);

            if (requestUrl != null)
            {
                buffer.Append("Request URL: ");
                buffer.Append(requestUrl.AbsoluteUri);

                buffer.Append(Environment.NewLine);
                buffer.Append(Environment.NewLine);
            }

            Exception baseException = ex.GetBaseException();
            Type baseExceptionType = baseException.GetType();

            buffer.Append("Exception Details: ");
            buffer.Append(baseExceptionType.FullName);
            buffer.Append(": ");
            buffer.Append(baseException.Message);

            buffer.Append(Environment.NewLine);
            buffer.Append(Environment.NewLine);

            buffer.Append("Stack Trace:");
            buffer.Append(Environment.NewLine);
            buffer.Append(Environment.NewLine);

            AppendExceptionDetail(ex, buffer);

            string message = buffer.ToString();
            Log(TraceEventType.Error, message);
        }

        /// <summary>
        /// Logs an error event to the trace listeners.
        /// </summary>
        /// <param name="message">The message to log.</param>
        public static void LogError(
            string message)
        {
            Log(TraceEventType.Error, message);
        }

        /// <summary>
        /// Logs an informational event to the trace listeners using the specified
        /// format string and arguments.
        /// </summary>
        /// <param name="provider">An System.IFormatProvider that supplies
        /// culture-specific formatting information.</param>
        /// <param name="format">A composite format string.</param>
        /// <param name="args">An System.Object array containing zero or more
        /// objects to format.</param>
        public static void LogInfo(
            IFormatProvider provider,
            string format,
            params object[] args)
        {
            string message = string.Format(
                provider, format, args);

            LogInfo(message);
        }

        /// <summary>
        /// Logs an informational event to the trace listeners.
        /// </summary>
        /// <param name="message">The message to log.</param>
        public static void LogInfo(
            string message)
        {
            Log(TraceEventType.Information, message);
        }

        /// <summary>
        /// Logs a warning event to the trace listeners using the specified
        /// format string and arguments.
        /// </summary>
        /// <param name="provider">An System.IFormatProvider that supplies
        /// culture-specific formatting information.</param>
        /// <param name="format">A composite format string.</param>
        /// <param name="args">An System.Object array containing zero or more
        /// objects to format.</param>
        public static void LogWarning(
            IFormatProvider provider,
            string format,
            params object[] args)
        {
            string message = string.Format(
                provider, format, args);

            LogWarning(message);
        }

        /// <summary>
        /// Logs a warning event to the trace listeners.
        /// </summary>
        /// <param name="message">The message to log.</param>
        public static void LogWarning(
            string message)
        {
            Log(TraceEventType.Warning, message);
        }
    }
}
```

In a [follow-up post](/blog/jjameson/2010/03/20/error-handling-in-moss-2007-applications), I'll cover how to handle errors in SharePoint Web applications         without necessarily adding a bunch or try/catch blocks, while still avoiding the         out-of-the-box SharePoint error page (which doesn't look very good on an Internet-facing         site).

> **Update (2011-01-31)**
>
> I've attached a sample Visual Studio solution that demonstrates the **Logger**
> class.

