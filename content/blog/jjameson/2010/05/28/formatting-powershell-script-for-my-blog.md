---
title: Formatting PowerShell Script for My Blog
date: 2010-05-28T08:31:00-06:00
excerpt:
  In a previous post , I described how I format code for my blog (i.e. by
  copying it from Visual Studio or SQL Server Management Studio and then running
  a simple console application to convert the RTF clipboard text to HTML). I
  recently started doing...
aliases:
  [
    "/blog/jjameson/archive/2010/05/27/formatting-powershell-script-for-my-blog.aspx",
    "/blog/jjameson/archive/2010/05/28/formatting-powershell-script-for-my-blog.aspx",
  ]
draft: true
categories: ["My System"]
tags: ["My System", "PowerShell"]
msdnBlogUrl: "http://blogs.msdn.com/b/jjameson/archive/2010/05/28/formatting-powershell-script-for-my-blog.aspx"
---

> **Note**
>
> This post originally appeared on my MSDN blog:
>
> [http://blogs.msdn.com/b/jjameson/archive/2010/05/28/formatting-powershell-script-for-my-blog.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/05/28/formatting-powershell-script-for-my-blog.aspx)
>
> Since
> [I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft),
> I have copied it here in case that blog ever goes away.

In a [previous post](/blog/jjameson/2009/10/09/formatting-code-for-my-blog), I
described how I format code for my blog (i.e. by copying it from Visual Studio
or SQL Server Management Studio and then running a simple console application to
convert the RTF clipboard text to HTML).

I recently started doing some significant work with PowerShell and therefore I
wanted to have a similar method of formatting PowerShell scripts for various
blog posts. Note that the approach I currently use for other code snippets
wouldn't work very well with PowerShell scripts, primarily because Visual Studio
doesn't currently provide syntax highlighting for PowerShell scripts.

Consequently I went looking for a secondary solution for formatting PowerShell
scripts. A little searching on the Internet quickly led to a blog post by Lee
Holmes:

{{< reference title="More PowerShell Syntax Highlighting"
linkHref="http://www.leeholmes.com/blog/MorePowerShellSyntaxHighlighting.aspx" >}}

While Lee's script outputs nicely formatted script as HTML, I decided to modify
his approach a little. It's not that there's really anything wrong with Lee's
script. Rather, given the simplicity of parsing PowerShell scripts into various
tokens, I decided that it would be preferable (and not to mention a great
PowerShell exercise for me) to output semantic HTML that is subsequently
formatted with CSS instead of via inline **style** attributes.

Here is the updated script that I ended up with (I've also attached it to this
post for easier downloading):

```
################################################################################
# Set-ClipboardScriptHtmlBlock.ps1
#
# Copies the entire contents of the currently selected Windows PowerShell ISE
# editor window to the clipboard. The copied data can be pasted into any
# application that supports pasting in UnicodeText or HTML format. Text pasted
# in HTML format will be formatted according to the specified CSS styles.
#
# Originally based on the Set-ClipboardScript.ps1 developed by Lee Holmes:
#
#    http://www.leeholmes.com/blog/MorePowerShellSyntaxHighlighting.aspx
#
# Updated to output semantic HTML (instead of inline styles) and to apply
# formatting via CSS.
#
################################################################################

[CmdletBinding()]
param($path)

function Get-ScriptName
{
    $myInvocation.ScriptName
}

if($path -and ([Threading.Thread]::CurrentThread.ApartmentState -ne "STA"))
{
    PowerShell -NoProfile -STA -File (Get-ScriptName) $path
    return
}

$cssClassMappings = @{
    'Attribute' = 'attribute'
    'Command' = 'command'
    'CommandArgument' = 'commandArgument'
    'CommandParameter' = 'commandParameter'
    'Comment' = 'comment'
    'GroupEnd' = $null
    'GroupStart' = $null
    'Keyword' = 'keyword'
    'LineContinuation' = 'lineContinuation'
    'LoopLabel' = 'loopLabel'
    'Member' = 'member'
    'NewLine' = 'newLine'
    'Number' = 'number'
    'Operator' = 'operator'
    'Position' = 'position'
    'StatementSeparator' = 'statementSeparator'
    'String' = 'string'
    'Type' = 'userType'
    'Unknown' = $null
    'Variable' = 'variable'
}

$styles = "<style type='text/css'>
code .attribute {
 color: #2b91af;
}
code .command {
 color: #00f;
}
code .commandArgument {
 color: #8a2be2;
}
code .commandParameter {
 color: #000080;
}
code .comment {
 color: #008000;
}
code .keyword {
 color: #00f;
}
code .number {
 color: #800080;
}
code .operator {
 color: #666;
}
code .string {
 color: #a31515;
}
code .userType {
 color: #2b91af;
}
code .variable {
 color: #ff4500;
}
div.codeBlock, div.consoleBlock, div.logExcerpt {
 background-color: #F4F4F4;
 border: 1px solid gray;
 cursor: text;
 font-family: 'Courier New',courier,monospace;
 margin: 10px 2px;
 max-height: 250px;
 overflow: auto;
 padding: 4px;
 width: 97.5%;
}
div.codeBlock pre, div.logExcerpt pre {
 margin: 0;
}
</style>"

Add-Type -Assembly System.Web
Add-Type -Assembly PresentationCore

# Generate an HTML span and append it to HTML string builder
$currentLine = 1
function Append-HtmlSpan ($block, $tokenType)
{
    if (($tokenType -eq 'NewLine') -or ($tokenType -eq 'LineContinuation'))
    {
        if($tokenType -eq 'LineContinuation')
        {
            $null = $codeBuilder.Append('`')
        }

        $null = $codeBuilder.Append("`r`n")
        $SCRIPT:currentLine++
    }
    else
    {
        $block = [System.Web.HttpUtility]::HtmlEncode($block)

        if($tokenType -eq 'String')
        {
            $lines = $block -split "`r`n"
            $block = ""

            $multipleLines = $false
            foreach($line in $lines)
            {
                if($multipleLines)
                {
                    $block += "`r`n"

                    $SCRIPT:currentLine++
                }

                $newText = $line.TrimStart()
                $newText = " " * ($line.Length - $newText.Length) + $newText
                $block += $newText
                $multipleLines = $true
            }
        }

        $cssClass = $cssClassMappings[$tokenType]

        If ($cssClass -ne $null)
        {
            $null = $codeBuilder.Append(
                "<span class='$cssClass'>$block</span>")
        }
        Else
        {
            $null = $codeBuilder.Append($block)
        }
    }
}

function GetHtmlClipboardFormat($html)
{
    $header = @"
Version:1.0
StartHTML:0000000000
EndHTML:0000000000
StartFragment:0000000000
EndFragment:0000000000
StartSelection:0000000000
EndSelection:0000000000
SourceURL:file:///about:blank
<!DOCTYPE HTML PUBLIC `"-//W3C//DTD HTML 4.0 Transitional//EN`">
<HTML>
<HEAD>
<TITLE>HTML Clipboard</TITLE>
__STYLES__
</HEAD>
<BODY>
<!--StartFragment-->
__HTML__
<!--EndFragment-->
</BODY>
</HTML>
"@

    $header = $header.Replace("__STYLES__", $styles)

    $startFragment = $header.IndexOf("<!--StartFragment-->") +
        "<!--StartFragment-->".Length + 2
    $endFragment = $header.IndexOf("<!--EndFragment-->") +
        $html.Length - "__HTML__".Length
    $startHtml = $header.IndexOf("<!DOCTYPE")
    $endHtml = $header.Length + $html.Length - "__HTML__".Length
    $header = $header -replace "StartHTML:0000000000",
        ("StartHTML:{0:0000000000}" -f $startHtml)
    $header = $header -replace "EndHTML:0000000000",
        ("EndHTML:{0:0000000000}" -f $endHtml)
    $header = $header -replace "StartFragment:0000000000",
        ("StartFragment:{0:0000000000}" -f $startFragment)
    $header = $header -replace "EndFragment:0000000000",
        ("EndFragment:{0:0000000000}" -f $endFragment)
    $header = $header -replace "StartSelection:0000000000",
        ("StartSelection:{0:0000000000}" -f $startFragment)
    $header = $header -replace "EndSelection:0000000000",
        ("EndSelection:{0:0000000000}" -f $endFragment)
    $header = $header.Replace("__HTML__", $html)

    Write-Verbose $header
    $header
}

function Main
{
    $text = $null

    if($path)
    {
        $text = (Get-Content $path) -join "`r`n"
    }
    else
    {
        if (-not $psise.CurrentFile)
        {
            Write-Error 'No script is available for copying.'
            return
        }

        $text = $psise.CurrentFile.Editor.Text
    }

    trap { break }

    # Do syntax parsing.
    $errors = $null
    $tokens = [system.management.automation.psparser]::Tokenize($text,
        [ref] $errors)

    # Initialize HTML builder.
    $codeBuilder = new-object system.text.stringbuilder
    $SCRIPT:currentLine++

    # Iterate over the tokens and set the colors appropriately.
    $position = 0
    foreach ($token in $tokens)
    {
        if ($position -lt $token.Start)
        {
            $block = $text.Substring($position, ($token.Start - $position))
            $tokenType = 'Unknown'
            Append-HtmlSpan $block $tokenType
        }

        $block = $text.Substring($token.Start, $token.Length)
        $tokenType = $token.Type.ToString()
        Append-HtmlSpan $block $tokenType

        $position = $token.Start + $token.Length
    }

    # Copy console screen buffer contents to clipboard in two formats -
    # text and HTML.
    $code = $codeBuilder.ToString()

    $codeBlock =
        "<div class='codeBlock'><pre><code>" + $code + "</code></pre></div>"

    $html = GetHtmlClipboardFormat($codeBlock)

    $dataObject = New-Object Windows.DataObject
    $dataObject.SetText([string]$codeBlock, [Windows.TextDataFormat]"UnicodeText")

    $dataObject.SetText([string]$html, [Windows.TextDataFormat]"Html")

    [Windows.Clipboard]::SetDataObject($dataObject, $true)
}

. Main
```

Now, all I have to do is run the Set-ClipboardScriptHtmlBlock script from within
the Windows PowerShell ISE (with a different script window active) and the
contents of the active script are copied to the clipboard. From there I can
paste the clipboard contents into the source window of Expression Web (currently
[my blog editor of choice](/blog/jjameson/2009/09/12/expression-web-my-msdn-blog-and-now-team-foundation-server)).

I then specify rules similar to the following in the custom CSS for my blog:

```
code .attribute {
   color: #2b91af;
}
code .command {
   color: #00f;
}
code .commandArgument {
   color: #8a2be2;
}
code .commandParameter {
   color: #000080;
}
code .comment {
   color: #008000;
}
code .keyword {
   color: #00f;
}
code .number {
   color: #800080;
}
code .operator {
   color: #666;
}
code .string {
   color: #a31515;
}
code .userType {
   color: #2b91af;
}
code .variable {
   color: #ff4500;
}
div.codeBlock {
   background-color: #F4F4F4;
   border: 1px solid gray;
   cursor: text;
   font-family: 'Courier New',courier,monospace;
   margin: 10px 2px;
   max-height: 250px;
   overflow: auto;
   padding: 4px;
   width: 97.5%;
}
div.codeBlock pre {
   margin: 0;
}
```

Note that this CSS is also embedded in the generated HTML to support pasting
into other applications (e.g. Microsoft Word). Also note that I generalized the
class names for the CSS rules a little bit, just in case I later decide to
generate semantic markup for other types of code (e.g. C#).

If you want your PowerShell scripts to appear differently in your blog posts,
all you would need to do is tweak the CSS accordingly.

Kudos to Lee for providing such a solid foundation to build upon!
