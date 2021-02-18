---
title: "Using CSS sprites to improve site performance and user experience (a.k.a. Building TechnologyToolbox.com, part 11)"
date: 2012-01-14T23:02:25-07:00
excerpt: "In my previous post, I briefly mentioned how the \"Most Popular Posts\" section on the Technology Toolbox home page uses a CSS sprite to render the arrow image next to each list item. In this post, I explain more about how CSS sprites are used on the site, why they are valuable, and some caveats when using them."
aliases: ["/blog/jjameson/archive/2012/01/14/building-technologytoolbox-com-part-11.aspx"]
draft: true
categories: ["Development", "My System"]
tags: ["Web Development"]
---

In
[my previous post](/blog/jjameson/2012/01/05/building-technologytoolbox-com-part-10), I briefly mentioned how the **Most Popular Posts**
section on the Technology Toolbox home page uses a CSS sprite to render the
arrow image next to each list item.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Home.png"
alt="Technology Toolbox home page"
height="600"
width="538"
title="Figure 1: Technology Toolbox home page" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Home.png)

If you use Firebug to inspect one of the list items in the **Most Popular
Posts** section, you'll notice the CSS `background` property is set to:

    url("Images/list-item-sprite-1.0.png") no-repeat scroll -200px -137px transparent

> **Note**
>
> You could also use the IE Developer Tools to inspect the element in order to view the corresponding CSS rule, but I prefer the experience in Firebug instead (since, unlike IE, Firebug actually shows you the image in a tooltip when you hover over the URL in the CSS rule).

The "list item sprite" image is actually a 400x400 composite of multiple
icons used throughout the site.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_list-item-sprites-1.0.png"
alt="list-item-sprites-1.0.png"
height="400"
width="400"
title="Figure 2: list-item-sprite-1.0.png" >}}

Notice that the "dashed arrow" icon used in the **Most Popular Posts** section is actually the fifth image in the set (starting from the lower-left
corner). This explains the negative offsets specified in the CSS rule (specifically,
-200px -137px).

If you do a quick Internet search for *[CSS sprites](http://www.google.com/search?q=CSS+sprites)*, you
will find a number of articles that explain more about the fundamental concepts
of CSS sprites and why they are useful for optimizing site perfomance (due to
the greatly reduced number of HTTP requests).

There are a few things worth pointing out regarding the use of CSS sprites
on the Technology Toolbox site:

- Why are the images arranged in a diagonal line from the bottom-left
  corner to the top-right corner?
- Why is there all that white space between the icons?
- What happens when someone needs to add a new list item icon?
- Are there any other benefits to using CSS sprites (i.e. besides reducing
  HTTP requests)?

Let's tackle these one at a time...

### Why are the images arranged in a diagonal line?

If you look at some of the example CSS sprites in the following article...

{{< reference title="CSS Sprites: What They Are, Why They’re Cool, and How To Use Them" linkHref="http://css-tricks.com/css-sprites/" >}}

...you will notice some sites "munge" a bunch of different images (of various
sizes) into a single CSS sprite, whereas I have chosen to only combine the "list
item" icons into the sample CSS sprite shown in Figure 2.

The reason why I did this is to avoid potential issues when list items span
multiple lines. Imagine instead that all of the list item icons shown in Figure
2 were arranged in close proximity (say, for example, aligned vertically with
minimal white space between them). In other words, imagine the "minus" icon
were directly below the "dashed arrow" icon used in the **Most Popular
Posts** section.

Consequently, in cases where the list item spans two lines (e.g. **Upgrade Team Foundation Server 2008 to TFS 2010 (and SharePoint Server 2010)**)
the "minus" icon would appear below the "dashed arrow" icon (which would definitely
confuse users).

Aaron Barker explains more about this technique in the following blog post:

{{< reference title="Diagonal CSS Sprites" linkHref="http://www.aaronbarker.net/2010/07/diagonal-sprites/" >}}

> **Important**
>
> Not all of the CSS sprites used on the Technology Toolbox site use the diagonal layout. For example, the "radio button" images used in the "slider" control (i.e. to the right of the **SharePoint Architecture
> and Development** heading in Figure 1) are rendered using a 22x40 sprite. In this case, there's no need to avoid potential issues due to a variable height of the element the CSS rule applies to.

### Why is there all that white space between the icons?

If you look at the example sprite shown in Aaron's post, and compare it with
the CSS sprite that I created (i.e. Figure 2), you'll notice that I chose to
leave a good deal more white space around each icon than Aaron did.

The reason for this is that I prefer to keep the "math" simple, while still
allowing for some potentially larger list item icons. Specifically, by treating
each list item icon as if it were 50x50 pixels, I know that to show the 5th
icon in the set, I need to set the x-position of the background to (5 - 1) \*
(-50px) = -200px. Figuring out the y-position is slightly trickier, since I
may need to take into account the actual height of the icon that I want to show
(in order to center the icon vertically with the first line of each list item).
In this case, it turns out to be -137px.

To show the "quote" icon, the y-position is 0; to show the "tag" icon, the
y-position is -50px; to show the "curved arrow", the y-position is -95px. The
point is that I can use a formula to determine the background image positions
(perhaps within a few pixels), and then tweak the y-position (as necessary)
within in a matter of seconds using Firebug.

Note that the sprite is created by exporting a PNG file from Expression Design.
The following screenshot illustrates how I use a "Grid" layer (in a rather hideous
shade of orange, so that I remember to hide it before exporting the image) to
quickly determine where to place the icons within the overall image. [If you
click to enlarge the image, you can probably count the 50x50 pixels used for
each cell in the grid -- or you can simply take my word for it :-) ]

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_list-items-sprite-design.png"
alt="List items sprite (Expression Design)"
height="377"
width="600"
title="Figure 3: List items sprite (Expression Design)" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_list-items-sprite-design.png)

### What happens when someone needs to add a new list item icon?

Ah yes...the astute reader will have realized by now (or perhaps known it
*a priori*) that when I need to add a new icon to the sprite, using this
"diagonal layout" method definitely comes with an inherent penalty.

When this happens, I need to fire up Expression Design, expand the artboard
(and grid), add the new icon, and then export the updated image -- giving it
a new filename to avoid any issues with cached images. This should explain the
"1.0" portion in the current filename.

However, by adding the new image in the upper right corner, all of the previously
established <var>x</var> and <var>y</var> offsets are no longer valid. Yes,
it's a pain -- but keep in mind that the new offsets are very easy to calculate
(simply by subtracting an additional 50 pixels -- assuming I've added only one
new icon). It's also rather easy to do a search for the sprite filename across
the CSS files to determine which `background`
rules need to be updated.

Is this a big deal? No...at least not to me.

### Are there any other benefits to using CSS sprites (i.e. besides reducing

HTTP requests)?

Well, as a matter of fact, yes. [Otherwise, why whould I have bothered to
include this section?]

Take a look at the **Archives** section displayed on the various
blog pages in the site, such as the one shown in Figure 4.

{{< figure
src="https://www.technologytoolbox.com/blog/images/www_technologytoolbox_com/blog/jjameson/7/r_Technology-Toolbox-Blog-Home.png"
alt="Blog home page"
height="600"
width="374"
title="Figure 4: Blog home page" >}}

[See full-sized image.](/blog/images/www_technologytoolbox_com/blog/jjameson/7/o_Technology-Toolbox-Blog-Home.png)

Notice the use of the "plus" icon from the sprite shown in Figure 2. If you
click one of the years, the item expands to show the months within that year
(for which at least one post was created). As you would imagine, the "plus"
icon also changes to a "minus" icon to indicate that clicking the item again
will collapse the list.

To accomplish this, I use a little jQuery to toggle the visibility of the
nested list items as well as a couple of CSS classes (specifically, `expanded` and `expandable`) on the list item that
was clicked. These CSS classes determine whether the "plus" or "minus" icon
is shown.

If separate image files are used for the "plus" and "minus" icons, users
experience a subtle flashing effect the first time they expand one of the years
(because the "minus" icon has to be downloaded). By using a CSS sprite instead,
the user experience is improved (albeit a slight improvement) because the "minus"
icon has already been downloaded. Simply tweaking the x and y positions toggles
which icon is displayed.

I'll explain more about the jQuery expandable list used to render the
**Archives** section in a separate
[post](/blog/jjameson/2012/01/16/building-technologytoolbox-com-part-12).

