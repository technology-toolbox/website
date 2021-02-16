---
title: "Import website (IIS) logs into SQL Server using Log Parser and PowerShell"
date: 2012-02-28T23:26:57-07:00
excerpt: "Here's a little PowerShell script I whipped up to import the TechnologyToolbox.com log files into a SQL Server database for some \"quick and dirty\" analysis."
draft: true
categories: ["Development", "Infrastructure"]
tags: ["Infrastructure", "PowerShell", "SQL Server", "Toolbox", "Web Development"]
---

This past weekend, while researching some errors reported on the TechnologyToolbox.com
website, I wanted to analyze the IIS logs -- for example, to see how many requests
have come from specific IP addresses (such as those suspected of attempting
to hack the site).

As I described in
[a previous post](/blog/jjameson/2012/02/03/building-technologytoolbox-com-part-22), TechnologyToolbox.com currently uses Google Analytics to
provide numerous metrics and track trends. However, Google Analytics doesn't
always provide the answer you are looking for. In fact, if an HTTP request results
in an error (typically as a result of a failed hack attempt), then the Google
Analytics script may never called and the request won't even be logged in that
system (depending on the specific error and how the website is configured to
handle errors).

Fortunately, when I setup the TechnologyToolbox.com website at WinHost, I
turned on the option to make the raw IIS logs available. Consequently, there's
a hidden folder (only available via FTP and only by the website owner) in which
WinHost dumps a zip file each day containing the raw IIS log file from the previous
day.

I've used
[Microsoft Log Parser](http://www.microsoft.com/download/en/details.aspx?displaylang=en&id=24659) to analyze IIS log files in the past (on client projects).
For quick, ad hoc analysis of a small amount of data, executing queries directly
against the log file via the command line is tolerable.

However, if you are going to repeatedly perform a number of queries against,
say, a few hundred thousand log entries, then it is worth the effort to first
import this data into a database.

Note that Log Parser includes an option to automatically create a table in
a specified database and subsequently import the data from the log files. However,
I prefer to create the table beforehand -- in order to tweak the column names,
combine the "date" and "time" fields into a single DATETIME column, and add
some "NOT NULL" constraints.

Here is the script I use to create the table in SQL Server:

```
CREATE TABLE dbo.WebsiteLog
(
    LogFilename VARCHAR(255) NOT NULL,
    RowNumber INT NOT NULL,
    EntryTime DATETIME NOT NULL,
    SiteName VARCHAR(255) NOT NULL,
    ServerName VARCHAR(255) NOT NULL,
    ServerIpAddress VARCHAR(255) NOT NULL,
    Method VARCHAR(255) NOT NULL,
    UriStem VARCHAR(255) NOT NULL,
    UriQuery VARCHAR(255) NULL,
    Port INT NOT NULL,
    Username VARCHAR(255) NULL,
    ClientIpAddress VARCHAR(255) NOT NULL,
    HttpVersion VARCHAR(255) NOT NULL,
    UserAgent VARCHAR(255) NOT NULL,
    Cookie VARCHAR(255) NULL,
    Referrer VARCHAR(255) NULL,
    Hostname VARCHAR(255) NOT NULL,
    HttpStatus INT NOT NULL,
    HttpSubstatus INT NOT NULL,
    Win32Status INT NOT NULL,
    BytesFromServerToClient INT NOT NULL,
    BytesFromClientToServer INT NOT NULL,
    TimeTaken INT NOT NULL
)
```

To import the TechnologyToolbox.com log files into a SQL Server database
(CaelumDW), I whipped up a PowerShell script to automatically:

1. Extract (a.k.a. unzip) the log files in the /httplog folder (which I
   periodically FTP from the Production environment) and subsequently move
   the zip files to the /httplog/Archive folder.
2. Import the log files using the LogParser utility.
3. Remove the log files from the /httplog folder (to avoid inserting duplicate
   data the next time the script is run).

Here is the script in hopes it helps others who wish to import their log
files for subsequent analysis.

> **Note**
>
> In the past, I've worked on several "clickstream data warehousing" projects where we performed this kind of import as the first step in the ETL process (prior to transforming the data into a star schema and building OLAP cubes against them). This simple **WebsiteLog** table is certainly not meant to serve as a replacement for a more sophisticated analytics solution. Rather I'm sharing it here for those wishing to do "quick and dirty" analysis of their data.

### Import Website Log Files.ps1

```
$ErrorActionPreference = "Stop"

Import-Module Pscx -EA 0

function ExtractLogFiles(
    [string] $httpLogPath)
{
    If ([string]::IsNullOrEmpty($httpLogPath) -eq $true)
    {
        Throw "The log path must be specified."    
    }
    
    [string] $httpLogArchive = $httpLogPath + "\Archive"
    
    If ((Test-Path $httpLogArchive) -eq $false)
    {
        Write-Host "Creating archive folder for compressed log files..."
        New-Item -ItemType directory -Path $httpLogArchive | Out-Null
    }
    
    Write-Host "Extracting compressed log files..."
    
    Get-ChildItem $httpLogPath -Filter "*.zip" |
        ForEach-Object {
            Expand-Archive $_ -OutputPath $httpLogPath
            
            Move-Item $_.FullName $httpLogArchive
        }
}

function ImportLogFiles(
    [string] $httpLogPath)
{
    If ([string]::IsNullOrEmpty($httpLogPath) -eq $true)
    {
        Throw "The log path must be specified."    
    }

    [string] $logParser = "${env:ProgramFiles(x86)}" `
        + "\Log Parser 2.2\LogParser.exe"

    [string] $query = `
        [string] $query = `
        "SELECT" `
            + " LogFilename" `
            + ", RowNumber" `
            + ", TO_TIMESTAMP(date, time) AS EntryTime" `
            + ", s-sitename AS SiteName" `
            + ", s-computername AS ServerName" `
            + ", s-ip AS ServerIpAddress" `
            + ", cs-method AS Method" `
            + ", cs-uri-stem AS UriStem" `
            + ", cs-uri-query AS UriQuery" `
            + ", s-port AS Port" `
            + ", cs-username AS Username" `
            + ", c-ip AS ClientIpAddress" `
            + ", cs-version AS HttpVersion" `
            + ", cs(User-Agent) AS UserAgent" `
            + ", cs(Cookie) AS Cookie" `
            + ", cs(Referer) AS Referrer" `
            + ", cs-host AS Hostname" `
            + ", sc-status AS HttpStatus" `
            + ", sc-substatus AS HttpSubstatus" `
            + ", sc-win32-status AS Win32Status" `
            + ", sc-bytes AS BytesFromServerToClient" `
            + ", cs-bytes AS BytesFromClientToServer" `
            + ", time-taken AS TimeTaken" `
        + " INTO WebsiteLog" `
        + " FROM $httpLogPath\*.log"
        
    [string] $connectionString = "Driver={SQL Server Native Client 10.0};" `
        + "Server=BEAST;Database=CaelumDW;Trusted_Connection=yes;"
    
    [string[]] $parameters = @()
    
    $parameters += $query
    $parameters += "-i:W3C"
    $parameters += "-o:SQL"
    $parameters += "-oConnString:$connectionString"
    
    Write-Debug "Parameters: $parameters"
    
    Write-Host "Importing log files to database..."
    & $logParser $parameters
}

function RemoveLogFiles(
    [string] $httpLogPath)
{
    If ([string]::IsNullOrEmpty($httpLogPath) -eq $true)
    {
        Throw "The log path must be specified."    
    }
    
    Write-Host "Removing log files..."    
    Remove-Item ($httpLogPath + "\*.log")
}
    
function Main
{
    [string] $httpLogPath = "C:\inetpub\wwwroot\www.technologytoolbox.com\httplog"

    ExtractLogFiles $httpLogPath

    ImportLogFiles $httpLogPath
    
    RemoveLogFiles $httpLogPath
        
    Write-Host -Fore Green "Successfully imported log files."
}

Main
```

> **Note**
>
> The script assumes you have installed Log Parser to the default location (on a 64-bit environment). You'll also need to update the log file path and connection string (unless you just happen to have a SQL Server named **BEAST** and a database named **CaelumDW**).

After running the script, you should see something like this:

```
PS C:\NotBackedUp\Public\Toolbox\PowerShell> {{< kbd "& \".\Import Website Log Files.ps1\"" >}}
Creating archive folder for compressed log files...
Extracting compressed log files...
Importing log files to database...

Statistics:
-----------
Elements processed: 210943
Elements output:    210943
Execution time:     155.13 seconds (00:02:35.13)

Removing log files...
Successfully imported log files.
```

