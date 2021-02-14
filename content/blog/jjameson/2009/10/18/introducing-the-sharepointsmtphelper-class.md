---
title: "Introducing the SharePointSmtpHelper Class"
date: 2009-10-18T00:29:00+08:00
excerpt: "Continuing in the spirit of my previous posts for the SharePointPublishingHelper class and SharePointWebPartHelper class, I'd like to introduce another helper class that you may find useful when building solutions for Windows SharePoint Services (WSS..."
draft: true
categories: ["My System", "SharePoint"]
tags: ["My System", "MOSS 2007", "WSS v3"]
---

> **Note**
> 
> This post originally appeared on my MSDN blog:  
>   
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/10/18/introducing-the-sharepointsmtphelper-class.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/10/18/introducing-the-sharepointsmtphelper-class.aspx)
> 
> Since [I no longer work for Microsoft](/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx), I have copied it here in case that blog ever goes away.


Continuing in the spirit of my previous posts for the `SharePointPublishingHelper` class and `SharePointWebPartHelper` class, I'd like to introduce another helper class that you may find useful when building solutions for Windows SharePoint Services (WSS) v3 and Microsoft Office SharePoint Server (MOSS) 2007.

Consider the scenario where you need to send an e-mail message from a SharePoint application. Perhaps an unexpected error occurred and you want to notify the operations team, or perhaps you just want to send a confirmation message to a user who has just submitted some important information.

Whatever the case, you'd like to send an e-mail message using the SMTP server configured in SharePoint Central Administration for the farm. `SharePointSmtpHelper` makes it really easy to do this.

Note that there's really not much code to the `SharePointSmtpHelper` class -- thanks to the classes provided by the .NET Framework in the [System.Net.Mail](http://msdn.microsoft.com/en-us/library/system.net.mail.aspx) namespace. It may look like more code than you might expect, but that's just because of all the overloads of the `SendMessage` method that I provide.


    using System;
    using System.Net.Mail;
    
    using Microsoft.SharePoint;
    using Microsoft.SharePoint.Administration;
    
    namespace Fabrikam.Demo.CoreServices.SharePoint
    {
        /// <summary>
        /// Exposes static methods for sending e-mail messages from a SharePoint
        /// Web application using the Simple Mail Transfer Protocol (SMTP). The
        /// SMTP server must be configured in SharePoint Central Administration.
        /// This class cannot be inherited.
        /// </summary>
        /// <remarks>
        /// All methods of the <b>SharePointSmtpHelper</b> class are static and
        /// can therefore be called without creating an instance of the class.
        /// </remarks>
        public sealed class SharePointSmtpHelper
        {
            private static object lockObject = new object();
    
            private static string smtpServer;
            private static string defaultFromAddress;
            
            private SharePointSmtpHelper( ) { } // all members are static
    
            /// <summary>
            /// Sends an e-mail message using the outbound SMTP server and
            /// "From address" configured in SharePoint Central Administration.
            /// </summary>
            /// <param name="toRecipients">The recipients of the e-mail message.
            /// In other words, the addresses on the To line of the e-mail.</param>
            /// <param name="subject">The subject line for the e-mail message.
            /// </param>
            /// <param name="body">The e-mail message body.</param>
            public static void SendMessage(
                string toRecipients,
                string subject,
                string body)
            {
                SendMessage(
                    toRecipients,
                    subject,
                    body,
                    false,
                    null,
                    null);
            }
    
            /// <summary>
            /// Sends an e-mail message using the outbound SMTP server and
            /// "From address" configured in SharePoint Central Administration,
            /// specifying whether the body of the message is formatted using HTML.
            /// </summary>
            /// <param name="toRecipients">The recipients of the e-mail message.
            /// In other words, the addresses on the To line of the e-mail.</param>
            /// <param name="subject">The subject line for the e-mail message.
            /// </param>
            /// <param name="body">The e-mail message body.</param>
            /// <param name="isBodyHtml"><c>true</c> if the mail message body is
            /// formatted using HTML markup. <c>false</c> if the message is simply
            /// plain text.</param>
            public static void SendMessage(
                string toRecipients,
                string subject,
                string body,
                bool isBodyHtml)
            {
                SendMessage(
                    toRecipients,
                    subject,
                    body,
                    isBodyHtml,
                    null,
                    null);
            }
    
            /// <summary>
            /// Sends an e-mail message using the outbound SMTP server and
            /// "From address" configured in SharePoint Central Administration,
            /// specifying whether the body of the message is formatted using HTML
            /// and also including carbon copy (CC) recipients.
            /// </summary>
            /// <param name="toRecipients">The recipients of the e-mail message.
            /// In other words, the addresses on the To line of the e-mail.</param>
            /// <param name="subject">The subject line for the e-mail message.
            /// </param>
            /// <param name="body">The e-mail message body.</param>        
            /// <param name="isBodyHtml"><c>true</c> if the mail message body is
            /// formatted using HTML markup. <c>false</c> if the message is simply
            /// plain text.</param>
            /// <param name="ccRecipients">The carbon copy (CC) recipients of the
            /// e-mail message.</param>
            public static void SendMessage(
                string toRecipients,
                string subject,
                string body,
                bool isBodyHtml,
                string ccRecipients)
            {
                SendMessage(
                    toRecipients,
                    subject,
                    body,
                    isBodyHtml,
                    ccRecipients,
                    null);
            }
    
            /// <summary>
            /// Sends an e-mail message using the outbound SMTP server configured
            /// in SharePoint Central Administration for the farm.
            /// </summary>
            /// <param name="toRecipients">The recipients of the e-mail message.
            /// In other words, the addresses on the To line of the e-mail.</param>
            /// <param name="subject">The subject line for the e-mail message.
            /// </param>
            /// <param name="body">The e-mail message body.</param>        
            /// <param name="isBodyHtml"><c>true</c> if the mail message body is
            /// formatted using HTML markup. <c>false</c> if the message is simply
            /// plain text.</param>
            /// <param name="ccRecipients">The carbon copy (CC) recipients of the
            /// e-mail message.</param>
            /// <param name="from">The e-mail address of the sender.</param>
            public static void SendMessage(
                string toRecipients,
                string subject,
                string body,
                bool isBodyHtml,
                string ccRecipients,
                string from)
            {
                if (string.IsNullOrEmpty(toRecipients) == true)
                {
                    throw new ArgumentNullException("toRecipients");
                }
    
                toRecipients = toRecipients.Trim();
                if (string.IsNullOrEmpty(toRecipients) == true)
                {
                    throw new ArgumentException(
                        "In order to send an e-mail message, "
                            + " at least one recipient must be specified.",
                        "toRecipients");
                }
    
                if (string.IsNullOrEmpty(subject) == true)
                {
                    throw new ArgumentNullException("subject");
                }
    
                subject = subject.Trim();
                if (string.IsNullOrEmpty(subject) == true)
                {
                    throw new ArgumentException(
                        "In order to send an e-mail message, "
                            + " the subject must be specified.",
                        "subject");
                }
    
                if (string.IsNullOrEmpty(body) == true)
                {
                    throw new ArgumentNullException("body");
                }
    
                body = body.Trim();
                if (string.IsNullOrEmpty(body) == true)
                {
                    throw new ArgumentException(
                        "In order to send an e-mail message, "
                            + " the message body must be specified.",
                        "body");
                }
    
                EnsureSmtpSettings();
    
                // If the "from" address is not specified, default to the address
                // configured in SharePoint Central Administration.
                if (string.IsNullOrEmpty(from) == true)
                {
                    from = defaultFromAddress;
                }
    
                SmtpClient smtpClient = new SmtpClient(smtpServer);
    
                MailMessage message = new MailMessage(
                    from,
                    toRecipients,
                    subject,
                    body);
    
                message.IsBodyHtml = isBodyHtml;
    
                if (string.IsNullOrEmpty(ccRecipients) == false)
                {
                    message.CC.Add(ccRecipients);
                }
    
                smtpClient.Send(message);
            }
    
            private static void EnsureSmtpSettings()
            {
                if (string.IsNullOrEmpty(smtpServer) == true)
                {
                    if (SPContext.Current == null)
                    {
                        throw new InvalidOperationException(
                            "SharePointSmtpClient can only be called from within"
                                + " a SharePoint site (SPContext.Current is null).");
                    }
    
                    lock (lockObject)
                    {
                        if (string.IsNullOrEmpty(smtpServer) == true)
                        {
                            SPWebApplication webApp =
                                SPContext.Current.Site.WebApplication;
    
                            if (webApp.OutboundMailServiceInstance != null
                                && webApp.OutboundMailServiceInstance.Server != null)
                            {
                                smtpServer =
                                    webApp.OutboundMailServiceInstance.Server.Address;
                            }
    
                            if (string.IsNullOrEmpty(smtpServer) == true)                            
                            {
                                throw new InvalidOperationException(
                                    "The outbound SMTP server is not specified in"
                                    + " SharePoint Central Administration.");
                            }
    
                            defaultFromAddress = webApp.OutboundMailSenderAddress;
                        }
                    }
                }
            }
        }
    }


Note that I use static members in order to only read the SMTP server configuration from SharePoint Central Administration one time.

Assuming you just want to send a simple message using the default "From address" specified in SharePoint Central Administration, you only need a single line of code that passes three parameters:


    SharePointSmtpHelper.SendMessage(
                    "jeremy_jameson@fabrikam.com",
                    "Test message",
                    "Testing the SharePointSmtpHelper class");

