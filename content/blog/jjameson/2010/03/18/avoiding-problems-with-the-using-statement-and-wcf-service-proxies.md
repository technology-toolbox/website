---
title: "Avoiding Problems with the Using Statement and WCF Service Proxies"
date: 2010-03-18T09:08:00-06:00
excerpt: "I encountered a rather nasty issue yesterday on my current project -- a customer portal built on Microsoft Office SharePoint Server (MOSS) 2007 that integrates with multiple external systems via Web services. 
 The database indexes for one of the external..."
aliases: ["/blog/jjameson/archive/2010/03/17/avoiding-problems-with-the-using-statement-and-wcf-service-proxies.aspx", "/blog/jjameson/archive/2010/03/18/avoiding-problems-with-the-using-statement-and-wcf-service-proxies.aspx"]
draft: true
categories: ["My System", "Development"]
tags: ["Simplify", "WCF"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/03/18/avoiding-problems-with-the-using-statement-and-wcf-service-proxies.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/03/18/avoiding-problems-with-the-using-statement-and-wcf-service-proxies.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

I encountered a rather nasty issue yesterday on my current project -- a customer
portal built on Microsoft Office SharePoint Server (MOSS) 2007 that integrates
with multiple external systems via Web services.

The database indexes for one of the external systems that we use to display data
on the portal somehow got corrupted. Consequently we encountered an error when
trying to view the corresponding portal page.

Fortunately, this didn't happen in the Production environment -- only the Test
environment -- and therefore only the Development and Test teams were effected
(no actual customers).

Unfortunately, the crux of the issue was not readily apparent due to an issue
with the proxy class used to interface with the Web service.

Being good .NET 3.0 citizens, we are using Windows Communication Foundation
(WCF) to generate the service proxies for accessing various Web services. In
other words, we are using "Service References" instead of the older "Web
References" from the original days of the .NET Framework.

Being good Web developers, we also try to hide the details about errors from end
users and instead log the exception details to the Windows event log and show a
generic error message on the page where the error occurred.

Shortly after starting to troubleshoot the problem, I discovered the following
error in the event log:

{{< blockquote "font-italic text-danger" >}}

The communication object, System.ServiceModel.Channels.ServiceChannel, cannot be
used for communication because it is in the Faulted state.

{{< /blockquote >}}

I also noticed from the stack trace that the error occurred while calling the
**Dispose** method of the Web service proxy class.

After a
[quick Internet search](http://www.bing.com/search?q=Dispose+%22cannot+be+used+for+communication+because+it+is+in+the+Faulted+state%22&form=QBRE&qs=n),
I discovered that there's a well-known problem with WCF service proxies and the
`using` statement in C#. [It obviously wasn't well-known to me until yesterday.]

There are a number of blog posts that describe the issue as well as various
hacks around the problem. However, the following MSDN article is the one that
really captured my attention:

{{< reference title="Avoiding Problems with the Using Statement"
linkHref="http://msdn.microsoft.com/en-us/library/aa355056.aspx" >}}

Here's the first sentence from that article:

{{< blockquote "font-italic" >}}

This sample demonstrates how you should not use the C# "using" statement to
automatically clean up resources when using a typed client.

{{< /blockquote >}}

Huh?

I have to admit, I was completely dumbfounded by this statement. An MSDN article
telling me that I should not use the C# "using" statement on an object that
implements
[IDisposable](http://msdn.microsoft.com/en-us/library/system.idisposable.aspx)?

What's that old movie quote about "dogs and cats living together" and "mass
hysteria"? According to my calendar, it's definitely still March -- so no April
Fool's joke here.

So, seriously, what gives? We are using "using" statements throughout our
solution. In my mind, this is a best practice. Heck, I even blogged once about
how I'm still waiting for that
[Utopian FxCop rule that barks at you whenever an object that implements IDisposable is not wrapped in a `using` block](/blog/jjameson/2009/03/19/to-dispose-or-not-to-dispose-that-is-the-question).

That's when it occurred to me: "Here we go again..."

It seems -- at least in my mind -- that there's a bug in
[System.ServiceModel.ClientBase](http://msdn.microsoft.com/en-us/library/ms576141.aspx),
and instead of admitting it's a bug and fixing the fundamental problem (which I
know can be painful), some "prescriptive guidance" is put out on MSDN on how to
workaround the bug.

Honestly, I can't see how this is any different than not properly freeing memory
in
[IDisposables containing IDisposables](/blog/jjameson/2008/04/09/memory-leak-in-splimitedwebpartmanager-a-k-a-idisposables-containing-idisposables)
and instead forcing the caller to cleanup the memory.

The problem -- at least in my opinion -- is that the **Dispose** method in
**ClientBase** is currently implemented as follows:

```
        void IDisposable.Dispose()
        {
            this.Close();
        }
```

However, as noted in the above MSDN article (and corresponding code sample),
this can cause problems when the C# "using" statement is used on the WCF service
proxy:

```
        using (CalculatorClient client = new CalculatorClient())
        {
            // Call Divide and catch the associated Exception.  This throws because the
            // server aborts the channel before returning a reply.
            try
            {
                client.Divide(0.0, 0.0);
            }
            catch (CommunicationException e)
            {
                Console.WriteLine("Got {0} from Divide.", e.GetType());
            }
        }

        // The previous line calls Dispose on the client.  Dispose and Close are the
        // same thing, and the Close is not successful because the server Aborted the
        // channel.  This means that the code after the using statement does not run.
        Console.WriteLine("Hope this code wasn't important, because it might not happen.");
```

The problem is that if an error occurs while calling the Web service, then when
`this.Close()` is subsequently called from the **Dispose** method, a
[**CommunicationObjectFaultedException**](http://msdn.microsoft.com/en-us/library/system.servicemodel.communicationobjectfaultedexception.aspx)
is thrown ("The communication object,
System.ServiceModel.Channels.ServiceChannel, cannot be used for communication
because it is in the Faulted state.").

The recommendation -- at least according to the above MSDN article -- is to
forego the C# "using" statement and instead rely on try/catch blocks (and
explicitly calling `client.Close()` or `client.Abort()` where appropriate.
[Excuse me while I go get a drink, because I've got this really bad taste in my
mouth right now.]

I completely understand the
[recommendation to catch TimeoutExceptions and CommunicationExceptions](http://msdn.microsoft.com/en-us/library/aa354510.aspx)
when calling Web services. However, that shouldn't necessarily mandate that I
cleanup the service proxy in various `catch` blocks -- or a `finally` block. If
a class implements **IDisposable**, then I want to call `Dispose()` on it!
[Well, actually, I don't want to explicitly call `Dispose()` -- what I really
want is to wrap the object in a `using` block.]

Wouldn't it be great if ClientBase actually implemented the **Dispose** method
as follows?

```
        void IDisposable.Dispose()
        {
            if (this.State == CommunicationState.Faulted)
            {
                this.Abort();
            }
            else if (this.State != CommunicationState.Closed)
            {
                this.Close();
            }
        }
```

Wouldn't this avoid the **CommunicationObjectFaultedException** that occurs when
the service proxy is wrapped in a `using` block, but an exception occurs when
calling the Web service?

Let's see...

How about creating a new "wrapper" class for the WCF service proxy?

```
using System;
using System.ServiceModel;

namespace Microsoft.ServiceModel.Samples
{
    public class CalculatorClientWithDisposeFix : CalculatorClient, IDisposable
    {
        #region IDisposable Members

        void IDisposable.Dispose()
        {
            if (this.State == CommunicationState.Faulted)
            {
                this.Abort();
            }
            else if (this.State != CommunicationState.Closed)
            {
                this.Close();
            }
        }

        #endregion
    }
}
```

With a couple of tweaks to the WCF sample code -- specifically replacing all
instances of "`new CalculatorClient()`" with "`new CalculatorClientWithDisposeFix()`" -- the output changes from this:

```
=
= Demonstrating problem:  closing brace of using statement can throw.
=
Got System.ServiceModel.EndpointNotFoundException from Divide.
Got System.ServiceModel.CommunicationObjectFaultedException
=
= Demonstrating problem:  closing brace of using statement can mask other Exceptions.
=
Got System.ServiceModel.EndpointNotFoundException from Divide.
Got System.ServiceModel.CommunicationObjectFaultedException
=
= Demonstrating cleanup with Exceptions.
=
Calling client.Add(0.0, 0.0);
Got System.ServiceModel.EndpointNotFoundException from Divide.
```

...to this:

```
=
= Demonstrating problem:  closing brace of using statement can throw.
=
Got System.ServiceModel.EndpointNotFoundException from Divide.
Hope this code wasn't important, because it might not happen.
=
= Demonstrating problem:  closing brace of using statement can mask other Exceptions.
=
Got System.ServiceModel.EndpointNotFoundException from Divide.
We do not come here because the ObjectDisposedException is masked.
=
= Demonstrating cleanup with Exceptions.
=
Calling client.Add(0.0, 0.0);
Got System.ServiceModel.EndpointNotFoundException from Divide.
```

The reality is that the WCF team might never fix this bug -- heck, for all I
know, this behavior could be marked as "by design." However, I'd like to think
that WCF service proxies will eventually be "fixed" (which is to say "function
as I -- and many other .NET developers -- would expect them to). I suppose
another option would be to remove the **IDisposable** interface from
**ClientBase** altogether (but I certainly don't like that idea).

And so continues my neverending quest to make the world of software a simpler --
and happier -- place to live in ;-)

