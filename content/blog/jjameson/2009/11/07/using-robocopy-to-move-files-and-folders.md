---
title: Using Robocopy to Move Files and Folders
date: 2009-11-07T05:46:00-07:00
excerpt:
  I use Robocopy a lot, and it's been in my Toolbox for so long that I hardly
  remember using anything else. I was glad to see that it is now included
  out-of-the-box (starting with Vista), because I typically use it instead of
  Windows Explorer to move or...
aliases:
  [
    "/blog/jjameson/archive/2009/11/06/using-robocopy-to-move-files-and-folders.aspx",
    "/blog/jjameson/archive/2009/11/07/using-robocopy-to-move-files-and-folders.aspx",
  ]
draft: true
categories: ["My System"]
tags: ["My System", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/11/07/using-robocopy-to-move-files-and-folders.aspx"
---

I use
[Robocopy](http://technet.microsoft.com/en-us/library/cc733145%28WS.10%29.aspx)
a lot, and it's been in my
[Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup) for so long that I
hardly remember using anything else. I was glad to see that it is now included
out-of-the-box (starting with Vista), because I typically use it instead of
Windows Explorer to move or copy files, since I prefer the way it reports the
progress of the lengthy file operations.

Note that when you specify the "/S /MOV" or "/E /MOV" options (to recursively
move files from one location to another), it may leave behind a bunch of empty
folders depending on the contents of the source and what you specify to move.

Over a year ago I went through the rather painful effort of organizing my
thousands of digital photos. During the process of moving thousands of files
around to various folders, I ended up with lots of empty folders in my source
folder structure.

As I discovered earlier this week, you can specify the /MOVE option (instead of
/MOV) to move both the files and directories, and delete them from the source
after they are copied.

Perhaps now my blog post from earlier this week about
[deleting empty folders](/blog/jjameson/2009/11/03/deleting-empty-folders) makes
a little more sense ;-)

Sometimes I'm amazed at what I learn while blogging. I had just been in such the
habit of using the /MOV option that I never gave it any thought (that is, until
this week).

