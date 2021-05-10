---
title: '"Could not get nonce" Error with pfSense ACME Certificates Service'
date: 2021-05-10T13:17:45-06:00
categories: ["Infrastructure"]
excerpt:
  Before you google "Could not get nonce" (like I did), you should first check
  the Let's Encrypt status page or Twitter account.
tags: ["Infrastructure", "pfSense"]
---

This morning, while creating a new [Let's Encrypt](https://letsencrypt.org/)
certificate using the Automated Certificate Management Environment (ACME)
package in pfSense, I encountered the following error:

{{< div-block "errorMessage" >}}

> Could not get nonce, let's try again.

{{< /div-block >}}

Actually, I encountered 20 of these errors:

{{< figure
  src="https://assets.technologytoolbox.com/screenshots/F6/CCD11176635DADB76FC53C0D50DD4F718ED0D6F6.png"
  class="screenshot" height="921" width="1024"
  title="Figure 1: Errors creating ACME (Let's Encrypt) certificate" >}}

[See full-sized image.](https://assets.technologytoolbox.com/screenshots/F6/CCD11176635DADB76FC53C0D50DD4F718ED0D6F6.png)

While investigating the issue, I stepped through the shell script that is
executed when you click the **Renew** button for a certificate -- starting with
running the following command (in an SSH session on the firewall):

```Shell
curl https://acme-v02.api.letsencrypt.org/directory -L --silent \
    --dump-header /tmp/acme/k8s-01.corp.technologytoolbox.com/http.header
```

This produced the following:

```Text
[Mon May 10 11:26:47 MDT 2021] response='{
  "type": "urn:acme:error:serverInternal",
  "detail": "The service is down for maintenance or had an internal error. Check https://letsencrypt.status.io/ for more details."
}'
```

Sure enough, when I checked the
[Let's Encrypt status page](https://letsencrypt.status.io/), I noticed the API
was down for planned maintenance -- which was also reflected in their Twitter
account:

{{< tweet 1391806837264060417 >}}

If you are using Let's Encrypt certificates, I recommend following that Twitter
account -- or at least checking the status page whenever you encounter an error
issuing or renewing certificates. Apparently the shell script used by pfSense is
somewhat lacking when it comes to handling planned maintenance on the API.
