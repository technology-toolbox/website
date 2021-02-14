---
title: "Internet Explorer 8 Upgrade"
date: 2009-03-24T02:12:00+08:00
excerpt: "In case you missed the announcement, Internet Explorer 8 was released last week. 
 I'll be honest, I tried one of the early betas of IE8 on a VM a long time ago, but I didn't spend a lot of time with it because it always seemed like there was some more..."
draft: true
categories: ["Development"]
tags: ["Web Development"]
---

> **Note**
> 
>             This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2009/03/24/internet-explorer-8-upgrade.aspx](http://blogs.msdn.com/b/jjameson/archive/2009/03/24/internet-explorer-8-upgrade.aspx)
> 
> 
> Since [I no longer work for Microsoft](/blog/jjameson/archive/2011/09/02/last-day-with-microsoft.aspx), I have copied it here in case that blog                 ever goes away.


In case you missed the announcement, [Internet Explorer 8](http://www.microsoft.com/windows/internet-explorer/default.aspx) was released last week.

I'll be honest, I tried one of the early betas of IE8 on a VM a long time ago, but         I didn't spend a lot of time with it because it always seemed like there was some         more pressing matter that required my attention. As I'm fond of saying, there are         typically three levels of issues that get piled on my plate: hot, hotter, and thermonuclear!

This past weekend, I upgraded 3 of my 10 *active* "machines" to IE8: my Vista         x64 desktop, my Windows Server 2008 x64 laptop, and a Windows Server 2008 x86 VM         that I am currently using as my SharePoint development environment. [I say *active*,         because this is the number of physical and virtual machines that I keep running         24x7 in the ["Jameson Datacenter"](/blog/jjameson/archive/2009/09/14/the-jameson-datacenter.aspx) -- a.k.a. my home office. No, I still haven't gotten         around to running that CAT5 cable from the second floor down to the basement so         I can get these portable space heaters -- er, I mean *servers* -- out of         the way.]

I'll wait for IE8 to be available via Windows Update before upgrading the remaining         active machines -- or the countless *inactive* machines in my inventory (i.e.         VMs that haven't been fired up in months or more, but I still keep around for demos         and reference material) if and when these are ever fired up again.

The upgrade from IE7 to IE8 on each of these three machines went without the slightest         glitch, and while I can't say that I'm heavily leveraging new features like Accelerators         and Web Slices (yet), I must admit that I really like the fact that IE8 is much         more Web standards compliant than any previous version of Internet Explorer.

As I've [hinted
            in the past](/blog/jjameson/archive/2008/10/20/fessing-up-about-firefox.aspx), the death of IE6 cannot come soon enough. I'm certainly not         the only person to have this opinion. Here are just a few of the dozens of sites         dedicated to eradicating IE6:

- [http://www.stoplivinginthepast.com](http://www.stoplivinginthepast.com/)
- [http://www.bringdownie6.com](http://www.bringdownie6.com/)
- [http://ie6.forteller.net/index.php?title=Main\_Page](http://ie6.forteller.net/index.php?title=Main_Page)
- [http://www.stopie6.com](http://www.stopie6.com/)
- [http://www.stopie6.org](http://www.stopie6.org/)
- [http://www.iedeathmarch.org](http://www.iedeathmarch.org/)


With IE8, I'm also digging the fact that tabs are grouped by color to indicate which         tabs were opened from a particular page. I have a habit of doing an Internet search         and then using <kbd>CTRL+</kbd>click to quickly open the top 3-5 results so I can         scan them for the information I am looking for. By highlighting the tabs with various         colors, it gives this scenario an improved user experience (for example, it is very         easy to quickly close tabs that I no longer want open).

I've also started using some of the [IE8 Developer Tools](http://msdn.microsoft.com/en-us/library/dd565628%28VS.85%29.aspx) -- and I must say these are a vast improvement over         the previous IE Developer Toolbar. However, it's much too early to say whether you'll         be able to pry [Firefox and its various add-ons](/blog/jjameson/archive/2008/10/20/fessing-up-about-firefox.aspx) out of my toolbox ;-)


> **Update (2009-03-25)**
> 
> 
> Yesterday I forgot to mention another vastly improved feature in IE8.
> 
> I tend to use <kbd>CTRL+F</kbd> frequently to find a particular string on a page                 (error messages, method names, etc.). When compared with Firefox, the user experience                 of the "Find on this page" feature in versions prior to IE8 left a little to be                 desired -- to put it mildly.
> 
> Prior to IE8, your could <kbd>CTRL+F</kbd>, start typing your search term, and then                 press <kbd>Enter</kbd> to quickly find the specified string on the page; however,                 if you decided that you wanted to search for something else, then a subsequent <kbd>                    CTRL+F</kbd> would only cause the **Find** dialog window to get                 focus, but you couldn't immediately start typing your new search term (because the                 focus would be on the **Next **button).
> 
> Fortunately, this has been overhauled in IE8 by eliminating the **Find**                 dialog altogether and instead integrating this feature into the main window. You                 can now quickly use <kbd>CTRL+F</kbd> repeatedly to search for different terms on                 a page. IE8 even has "hit highlighting" similar to Firefox.

