---
title: "Showing Resized Images on a Community Server/Telligent Blog"
date: 2010-06-07T07:57:00-06:00
excerpt: "Back in Community Server 2.1 (i.e. prior to the recent upgrade of the MSDN/TechNet blog platform), I could specify a URL like the following to show a smaller version of an image in a blog post: 
 http://blogs.msdn.com/photos/jjameson/images/9997719/500x258..."
aliases: ["/blog/jjameson/archive/2010/06/06/showing-resized-images-on-a-community-server-telligent-blog.aspx", "/blog/jjameson/archive/2010/06/07/showing-resized-images-on-a-community-server-telligent-blog.aspx"]
draft: true
categories: ["My System"]
tags: ["My System"]
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/06/07/showing-resized-images-on-a-community-server-telligent-blog.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/06/07/showing-resized-images-on-a-community-server-telligent-blog.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

Back in Community Server 2.1 (i.e. prior to the recent upgrade of the
MSDN/TechNet blog platform), I could specify a URL like the following to show a
smaller version of an image in a blog post:

> [http://blogs.msdn.com/photos/jjameson/images/9997719/500x258.aspx](http://blogs.msdn.com/photos/jjameson/images/9997719/500x258.aspx)

I also provided a link to allow users to see the full-size image, for example:

> [http://blogs.msdn.com/photos/jjameson/images/9997719/original.aspx](http://blogs.msdn.com/photos/jjameson/images/9997719/original.aspx)

Here's some typical HTML markup for showing images in my blog posts:

```
    <div class="image">
        <img src="http://blogs.msdn.com/photos/jjameson/images/9997719/500x258.aspx"
            alt="Hyper-V Manager showing the staggered start of various VMs"
            width="500" height="258" />
        <div class="caption">
            Figure 1: Hyper-V staggered start</div>
        <div class="imageLink">
            <a href="http://blogs.msdn.com/photos/jjameson/images/9997719/original.aspx"
            target="_blank">
                See full-sized image.</a>
        </div>
    </div>
```

After the upgrade of the MSDN/TechNet blog platform to Telligent Community 5.5,
we can no longer store images in "/photos" (i.e. media libraries have been
disabled in our Telligent installation). Instead, we are now expected to upload
images through the file storage feature on our blogs (i.e. via our blog
dashboard pages, by clicking **Manage** --&gt; **Files**).

Note that before the upgrade, it was very easy to grab the URLs of resized
images stored in Community Server photo galleries (now called media libraries, I
believe). However, when uploading images through blog file storage, you aren't
provided with any out-of-the-box functionality to view the images in various
sizes.

Fortunately, with a little extra work, you can use one of the out-of-the-box
HTTP handlers (resized-image.ashx) to dynamically resize images uploaded via
blog file storage in Telligent Community 5.5.

For example, the new URL corresponding to the first URL shown above is:

> [http://blogs.msdn.com/resized-image.ashx/\_\_size/500x258/\_\_key/CommunityServer-Components-PostAttachments/00-09-99-77-19/Hyper\_2D00\_V-Staggered-Start.png](http://blogs.msdn.com/resized-image.ashx/__size/500x258/__key/CommunityServer-Components-PostAttachments/00-09-99-77-19/Hyper_2D00_V-Staggered-Start.png)

Essentially it's a matter of replacing "cfs-filesystemfile.ashx" in the URL with
"resized-image.ashx" and adding the desired size for the image.

Note that you don't have to specify both height and width for the new image
size. For example, "/\_\_size/500x0" produces similar results in the above
example (in other words, the height is automatically calculated in order to
preserve the aspect ratio of the image).

> **Important**
>
> There appears to be a bug in Telligent Evolution 5.5 when the filename for the
> image contains certain characters (e.g. parentheses). For example, if you
> upload an image named "Upgraded TFS project site in SharePoint Server 2010
> (with Reporting Services error).png"and then attempt to dynamically resize it
> using the resized-image.ashx handler, you will find it simply doesn't work.
> Fortunately if you rename the image to remove the parentheses (and
> subsequently upload the renamed file), then the image resizer works as
> expected.

