---
title: Formatting Code for My Blog
date: 2009-10-09T06:46:00-06:00
description:
  "It occurred to me this morning that while I previously shared some details on
  how I manage my MSDN blog , I've never shared my method for formatting code
  for the Web. Actually, calling it \"my method\" is definitely a bit of a
  stretch. I certainly didn..."
aliases:
  [
    "/blog/jjameson/archive/2009/10/08/formatting-code-for-my-blog.aspx",
    "/blog/jjameson/archive/2009/10/09/formatting-code-for-my-blog.aspx",
  ]
categories: ["My System", "Development"]
tags: ["My System", "Simplify", "Visual Studio", "Web Development", "Toolbox"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2009/10/09/formatting-code-for-my-blog.aspx"
---

It occurred to me this morning that while I previously shared some details on
[how I manage my MSDN blog](/blog/jjameson/2009/09/12/expression-web-my-msdn-blog-and-now-team-foundation-server),
I've never shared my method for formatting code for the Web.

Actually, calling it "my method" is definitely a bit of a stretch. I certainly
didn't come up with the approach but rather refined someone else's approach (and
code) to suit my needs. The original credit goes to
[David Anson](http://blogs.msdn.com/delay).

Prior to adopting David's approach, I previously copied code from Visual Studio,
pasted it into Microsoft Word, then copied it again in Microsoft Word, and
finally pasted into Expression Web. While this achieved the desired end result
-- meaning, formatted code with color syntax highlighting -- it definitely was a
little kludgey. First, the four-step process -- while not excessively tedious --
was definitely less than desirable, because it often meant I had to fire up Word
for the sole purpose of formatting code (note that copy/paste from Visual Studio
into Expression Web does not preserve the formatting). Second -- and more
importantly -- the resulting HTML markup was just "crap" (sorry, I can't think
of a more eloquent way to put that right now).

When I started looking at approaches for improving the formatting of code for my
blog, I found that there were numerous "solutions" out there. However, I knew
what I really wanted was something that would produce a minimal amount of HTML
markup (or at least a lot less HTML than Microsoft Word) and, preferably,
something that generated semantic markup as well -- or at least _reasonably_
semantic markup.

What I mean by semantic markup is that code should be wrapped in a `<code>` tag
-- as well as a `<pre>` tag (in order to correctly display line breaks,
indenting, etc.). To understand what I mean by semantic markup for code,
consider the following example:

```C#
using System;

public class Class1
{
    public Class1()
    {
        string foo = "foo";
    }
}
```

In HTML markup, this is expressed as:

```HTML
<div class='codeBlock'><pre><code><span style='color:#0000ff'>using</span> System;

<span style='color:#0000ff'>public</span> <span style='color:#0000ff'>class</span> <span style='color:#2b91af'>Class1
</span>{
    <span style='color:#0000ff'>public</span> Class1()
    {
        <span style='color:#0000ff'>string</span> foo = <span style='color:#a31515'>"foo"</span>;
    }
}</code></pre></div>
```

The `<div class='codeBlock'>` element is used to constrain lengthy code blocks
(i.e. show a vertical scrollbar when necessary) and also format code with a
background color and border.

Note that in the stricted sense, this isn't 100% semantic markup because the
`<span>` tags are used to apply presentational styles, namely the various font
colors. Truly semantic markup for code would specify something more like this:

```HTML
<div class='codeBlock'><pre><code><span class='keyword'>using</span> System;

<span class='keyword'>public</span> <span class='keyword'>class</span> <span class="userType">Class1
</span>{
    <span class='keyword'>public</span> Class1()
    {
        <span class='keyword'>string</span> foo = <span class='string'>"foo"</span>;
    }
}
</code></pre></div>
```

CSS rules could then be used to achieve the color syntax highlighting:

```CSS
code .keyword
{
    color: #0000ff;
}
code .string
{
    color: #a31515;
}
code .userType
{
    color: #2b91af;
}
```

Anyway, getting back to the real topic for this post...

I really liked David's approach since a) it was simple (meaning I didn't need to
spend much time understanding his code sample), and b) it leveraged Visual
Studio to handle the bulk of the formatting work.

However, there were a couple of "fixes" that I found I needed:

- David's original code sample produced errors when I used it from SQL Server
  Management Studio 2005 (note that SQL Server Management Studio uses the Visual
  Studio "shell", so one should expect this to work)
- David's code sample only inserted a `<pre>` tag (and not the corresponding
  `<code>` and `<div class="codeBlock">` tags that I also wanted)

I also wanted to eliminate extraneous `<span>` tags for the default color (i.e.
black).

Note that -- at least to this point -- I haven't made any attempt to generate
100% semantic markup (by converting the `style` attributes to corresponding CSS
class names), because I think it would be somewhat brittle (i.e. dependent on
the default color options in Visual Studio) and, honestly, not worth the effort.

{{< div-block "note update" >}}

> **Update (2010-04-27)**
>
> After upgrading to Visual Studio 2010, I discovered that "\par" tags in the
> RTF were not being converted to new lines in David's original code (because
> Visual Studio 2008 didn't emit "\par" tags in the RTF, but rather "\r\n").
> Consequently I updated the code below to work with Visual Studio 2010 (similar
> to the changes described in
> [David's updated post from last December](http://blogs.msdn.com/delay/archive/2009/12/20/blogging-code-samples-stays-easy-update-to-free-convertclipboardrtftohtmltext-tool-and-source-code-for-visual-studio-2010.aspx)).

{{< /div-block >}}

Here's the updated code that I now use for my Rtf2Html.exe utility:

```C#
// Original source:
// http://blogs.msdn.com/delay/archive/2008/03/13/
// blogging-code-samples-should-be-easy-free-convertclipboardrtftohtmltext-tool-and-source-code.aspx

using System;
using System.Collections.Generic;
using System.Drawing;
using System.Text;
using System.Windows.Forms;

// Convert Visual Studio 2008 RTF clipboard format into HTML by replacing the
// clipboard contents with its HTML representation in text format suitable for
// pasting into a web page or blog.
// USE: Copy to clipboard in VS, run this app (no UI), paste converted text
// NOTE: This is NOT a general-purpose RTF-to-HTML converter! It works well
// enough on the simple input I've tried, but may break for other input.
// TODO: Convert into a real application with a notify icon and hotkey.
namespace ConvertClipboardRtfToHtmlText
{
    static class ConvertClipboardRtfToHtmlText
    {
        private const string colorTbl = "\\colortbl;";
        private const string colorFieldTag = "cf";
        private const string tabExpansion = "    ";

        [System.Diagnostics.CodeAnalysis.SuppressMessage(
            "Microsoft.Maintainability",
            "CA1502:AvoidExcessiveComplexity")]
        [STAThread]
        static void Main()
        {
            if (Clipboard.ContainsText(TextDataFormat.Rtf))
            {
                // Create color table, populate with default color
                List<Color> colors = new List<Color>();
                Color defaultColor = Color.FromArgb(0, 0, 0);
                colors.Add(defaultColor);

                bool insideSpan = false;

                // Get RTF
                string rtf = Clipboard.GetText(TextDataFormat.Rtf);

                // Parse color table
                int i = rtf.IndexOf(
                    colorTbl,
                    StringComparison.OrdinalIgnoreCase);

                if (-1 != i)
                {
                    i += colorTbl.Length;

                    // When copying from Visual Studio, we expect
                    // "\\colortbl;\r\n". However when copying from SQL Server
                    // Management Studio, we expect just "\\colortbl;".
                    SkipOptionalText(rtf, ref i, "\r\n");

                    while ((i < rtf.Length) && ('}' != rtf[i]))
                    {
                        // Add color to color table
                        SkipOptionalText(rtf, ref i, "\r\n");
                        SkipExpectedText(rtf, ref i, "\\red");
                        byte red = (byte)ParseNumericField(rtf, ref i);
                        SkipExpectedText(rtf, ref i, "\\green");
                        byte green = (byte)ParseNumericField(rtf, ref i);
                        SkipExpectedText(rtf, ref i, "\\blue");
                        byte blue = (byte)ParseNumericField(rtf, ref i);
                        colors.Add(Color.FromArgb(red, green, blue));
                        SkipOptionalText(rtf, ref i, "\r\n");
                        SkipExpectedText(rtf, ref i, ";");
                    }
                }
                else
                {
                    throw new NotSupportedException(
                        "Missing/unknown colorTbl.");
                }

                // Find start of text and parse
                i = rtf.IndexOf("\\fs", StringComparison.OrdinalIgnoreCase);
                if (-1 != i)
                {
                    // Skip font size tag
                    while ((i < rtf.Length) && (' ' != rtf[i]))
                    {
                        i++;
                    }
                    i++;

                    // Begin building HTML text
                    StringBuilder sb = new StringBuilder();
                    sb.Append("<div class='codeBlock'><pre><code>");
                    while (i < rtf.Length)
                    {
                        if ('\\' == rtf[i])
                        {
                            // Parse escape code
                            i++;
                            if ((i < rtf.Length) &&
                                (('{' == rtf[i])
                                    || ('}' == rtf[i])
                                    || ('\\' == rtf[i])))
                            {
                                // Escaped '{' or '}' or '\'
                                sb.Append(rtf[i]);
                            }
                            else
                            {
                                // Parse tag
                                int tagEnd = rtf.IndexOf(' ', i);
                                if (-1 != tagEnd)
                                {
                                    if (rtf.Substring(
                                        i,
                                        tagEnd - i).StartsWith(
                                            colorFieldTag,
                                            StringComparison.OrdinalIgnoreCase))
                                    {
                                        // Parse color field tag
                                        i += colorFieldTag.Length;
                                        int colorIndex = ParseNumericField(rtf, ref i);
                                        if ((colorIndex < 0)
                                            || (colors.Count <= colorIndex))
                                        {
                                            throw new NotSupportedException(
                                                "Bad color index.");
                                        }

                                        if (insideSpan == true)
                                        {
                                            sb.Append("</span>");
                                            insideSpan = false;
                                        }

                                        // Change to new color
                                        if (colors[colorIndex] != defaultColor)
                                        {
                                            sb.AppendFormat(
                                                "<span style='color:#{0:x2}{1:x2}{2:x2}'>",
                                                colors[colorIndex].R, colors[colorIndex].G,
                                                colors[colorIndex].B);

                                            insideSpan = true;
                                        }
                                    }
                                    else if ("par" ==
                                        rtf.Substring(i, tagEnd - i))
                                    {
                                        sb.Append(Environment.NewLine);
                                    }
                                    else if ("tab" ==
                                        rtf.Substring(i, tagEnd - i))
                                    {
                                        sb.Append(tabExpansion);
                                    }

                                    // Skip tag
                                    i = tagEnd;
                                }
                                else
                                {
                                    throw new NotSupportedException(
                                        "Malformed tag.");
                                }
                            }
                        }
                        else if ('}' == rtf[i])
                        {
                            // Terminal curly; done
                            break;
                        }
                        else
                        {
                            // Normal character; HTML-escape '<', '>', and '&'
                            switch (rtf[i])
                            {
                                case '<':
                                    sb.Append("&lt;");
                                    break;
                                case '>':
                                    sb.Append("&gt;");
                                    break;
                                case '&':
                                    sb.Append("&amp;");
                                    break;
                                default:
                                    sb.Append(rtf[i]);
                                    break;
                            }
                        }
                        i++;
                    }

                    // Trim any trailing empty lines
                    while ((2 <= sb.Length)
                        && ('\r' == sb[sb.Length - 2])
                        && ('\n' == sb[sb.Length - 1]))
                    {
                        sb.Length -= 2;
                    }

                    // Finish building HTML text
                    if (insideSpan == true)
                    {
                        sb.Append("</span>");
                        insideSpan = false;
                    }

                    sb.Append("</code></pre></div>");

                    // Update the clipboard text
                    Clipboard.SetText(sb.ToString());
                }
                else
                {
                    throw new NotSupportedException(
                        "Missing text section.");
                }
            }
        }

        // Skip the specified text
        private static void SkipExpectedText(string s, ref int i, string text)
        {
            foreach (char c in text)
            {
                if ((s.Length <= i) || (c != s[i]))
                {
                    throw new NotSupportedException("Expected text missing.");
                }
                i++;
            }
        }

        private static void SkipOptionalText(
            string s,
            ref int i,
            string text)
        {
            string substring = s.Substring(i, text.Length);

            if (substring == text)
            {
                i += text.Length;
            }
        }

        // Parse a numeric field
        private static int ParseNumericField(string s, ref int i)
        {
            int value = 0;
            while ((i < s.Length) && char.IsDigit(s[i]))
            {
                value *= 10;
                value += s[i] - '0';
                i++;
            }
            return value;
        }
    }
}
```

I suppose I could have kept the ConvertClipboardRtfToHtmlText moniker that David
originally had, but for some reason I decided to abbreviate it. In hindsight,
I'm really not sure why I didn't keep the name, but oh well...

Whenever I need a code sample in a blog post, I simply copy the code in Visual
Studio, double-click Rtf2Html.exe in my
[Toolbox](/blog/jjameson/2007/03/22/backedup-and-notbackedup), and then paste
into Expression Web. It's still a three-step process, but not one that takes
more than three or four seconds, and more importantly, it now produces much
better HTML markup than the hack I used to use with Microsoft Word.
