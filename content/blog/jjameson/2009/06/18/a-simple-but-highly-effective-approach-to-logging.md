---
title: A Simple, but Highly Effective Approach to Logging
date: 2009-06-18T16:01:00-06:00
excerpt:
  A common question that frequently arises both with customers and fellow
  consultants is what do I recommend for logging? As experienced software
  developers, we know that there are going to be errors in our solution -- as
  well as other important events...
aliases:
  [
    "/blog/jjameson/archive/2009/06/18/a-simple-but-highly-effective-approach-to-logging.aspx",
  ]
draft: true
categories: ["My System", "SharePoint", "Development"]
tags: ["Simplify", "MOSS 2007", "Core Development", "WSS v3"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/06/18/a-simple-but-highly-effective-approach-to-logging.aspx"
---

A common question that frequently arises both with customers and fellow
consultants is what do I recommend for logging? As experienced software
developers, we know that there are going to be errors in our solution -- as well
as other important events that we want to monitor -- and therefore we know we
need a robust way of logging these.

However, with numerous options available -- e.g. log4net, Enterprise Library,
Common.Logging, SharePoint ULS, etc. -- it's no wonder this is frequently a
"hot" topic.

Several years ago, I created a simple, but highly effective approach to logging
based on the (then) new System.Diagnostics features in the .NET Framework
version 2.0.

Before introducing my logging feature, it is first important to understand the
goals (and non-goals) of the solution.

### Goals and Non-Goals

The primary goals of the logging feature are:

- Provide solution components a way of logging messages of various levels (e.g.
  errors, warnings, information) with minimal custom code.
- Ensure log messages can be enabled in all environments (e.g. DEV, TEST, and
  PROD); in other words, in both Debug and Release builds (unlike the ` Debug.WriteLine` method, which relies on the DEBUG conditional compilation
  constant).
- Through configuration, enable log messages to be routed to various outputs
  such as a text file, the Windows Event Log, or
  [ASP.NET tracing](http://msdn.microsoft.com/en-us/library/bb386420.aspx).
- Ensure that different levels of log messages can be filtered and potentially
  routed to different outputs; for example, log all messages to the console, but
  only log errors and warnings to a text file.

Non-goals of the logging feature include:

- Providing the fastest possible logging implementation; rather the performance
  impact of logging should be insignificant when compared with the "real" work
  performed by the solution components.
- Changing logging configuration without reinitializing the solution; for
  example, to change the logging for an ASP.NET application, it is acceptable to
  restart the corresponding application pool.
- Filtering log messages based on subsystems (or feature areas) of the solution;
  for example, when logging is enabled for debug messages, then log messages
  from all components are output.

### Introducing the Logger Class

With the custom `Logger` class, logging a debug message is simply a matter of
calling a static method, specifying nothing more than a string containing the
message :

```C#
    Logger.LogDebug("Successfully loaded search results into DataSet.");
```

This example shows how the `Logger` class achieves the primary design goal. Note
that there is no need to explicitly create objects within each class -- or
create additional classes within an assembly -- for logging purposes.

Also note that the `Logger` class provides additional overloads to easily format
log messages:

```C#
    Logger.LogDebug(
        CultureInfo.InvariantCulture,
        "Successfully loaded embedded resource ({0})"
            + " ({1:n0} bytes) from assembly ({2}).",
        resourceName,
        resourceContent.Length,
        resourceAssembly.FullName);
```

Note that the `Logger.LogDebug` method is simply a convenient alternative to the
`Logger.Log` method:

```C#
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
```

Other methods such as `LogInfo` and `LogError` provide similar overloads for
convenience.

The simplicity of the `Logger` class is made possible by the improved tracing
functionality introduced in the .NET Framework version 2.0. Specifically, the
`Logger` class is simply a
"[wafer-thin](http://en.wikipedia.org/wiki/Mr_Creosote)" wrapper around the
[System.Diagnostics.TraceSource](http://msdn.microsoft.com/en-us/library/system.diagnostics.tracesource%28VS.80%29.aspx)
class.

The `Logger` class declares a singleton `TraceSource` that is used to log all
messages:

```C#
    private static TraceSource defaultTraceSource =
        new TraceSource("defaultTraceSource");
```

Various listeners can then be configured to output log messages. Each type of
listener derives from
[TraceListener](http://msdn.microsoft.com/en-us/library/system.diagnostics.tracelistener%28VS.80%29.aspx).

Note that the .NET Framework includes listeners for logging to a file, the
Windows Event Log, as well as ASP.NET tracing. Consequently, with just a little
bit of custom code (i.e. the `Logger` class) combined with all the "goodness"
baked into the core .NET Framework, we have a logging feature that meets all of
the established goals.

In my
[next post](/blog/jjameson/2009/06/18/configuring-logging-in-a-console-application),
I introduce how to configure logging (starting out with a console application).

Here is the complete source for the `Logger` class:

```C#
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
    public sealed class Logger
    {
        private static TraceSource defaultTraceSource =
            new TraceSource("defaultTraceSource");

        private Logger() { } // all members are static

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

{{< div-block "note update" >}}

> **Update 2010-03-20**
>
> A newer version of the **Logger** class is available in the following post:
>
> {{< reference title="Logging Exceptions in .NET Applications" linkHref="/blog/jjameson/2010/03/20/logging-exceptions-in-net-applications" linkText="http://blogs.msdn.com/jjameson/archive/2010/03/20/logging-exceptions-in-net-applications.aspx" >}}

{{< /div-block >}}

Note that the Logger.cs file actually includes `#define TRACE` at the top of the
file. This is because I originally wrote this class with an old version of
Visual Studio (which did not define this compilation constant by default when
creating new projects). Visual Studio 2008 projects include this in the project
options by default (for both Debug and Release configurations), so this is
superfluous.

As you can see, there's really not much to this class (which hopefully means
there isn't much that can go wrong with it). Most, if not all, of it should be
very straightforward. I'll explain the importance of the `Logger.Flush` method
in my next post.
