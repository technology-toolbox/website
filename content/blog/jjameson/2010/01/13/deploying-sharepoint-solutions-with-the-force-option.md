---
title: "Deploying SharePoint Solutions with the \"-force\" Option"
date: 2010-01-13T00:19:00+08:00
excerpt: "In a previous post , I provided sample \"DR.DADA\" scripts that I use for deploying solutions based on Microsoft Office SharePoint Server (MOSS) 2007. 
 If you've read that post, you might recall seeing the following lines in, for example, the Deploy Solutions..."
draft: true
categories: ["SharePoint"]
tags: ["MOSS 2007", "WSS v3"]
---

> **Note**
> 
> 
> 	This post originally appeared on my MSDN blog:  
>   
> 
> 
> [http://blogs.msdn.com/b/jjameson/archive/2010/01/13/deploying-sharepoint-solutions-with-the-force-option.aspx](http://blogs.msdn.com/b/jjameson/archive/2010/01/13/deploying-sharepoint-solutions-with-the-force-option.aspx)
> 
> 
> Since
> 	[I no longer work for Microsoft](/blog/jjameson/2011/09/02/last-day-with-microsoft), I have copied it here in case that blog 
> 	ever goes away.


In a [previous post](/blog/jjameson/2009/09/28/sample-walkthrough-of-the-dr-dada-approach-to-sharepoint), I provided sample "DR.DADA" scripts that I use for deploying  solutions based on Microsoft Office SharePoint Server (MOSS) 2007.

If you've read that post, you might recall seeing the following lines in, for  example, the Deploy Solutions.cmd script:

`REM Sometimes it is necessary to force the deployment to circumvent errorsREM set FORCE_OPTION=-force`

The FORCE\_OPTION environment variable is subsequently included in the line that  invokes StsAdm.exe:

<kbd>%SPDIR%\bin\stsadm.exe -o deploysolution -name "%SOLUTION_NAME%.wsp" -url %FABRIKAM_PORTAL_URL% %DEPLOY_METHOD% -allowGacDeployment %FORCE_OPTION%</kbd>

What's all this nonsense about forcing the deployment to "circumvent errors"?  Yes, it's admittedly a hack (although for some reason when I originally created  these scripts years ago, I didn't label it as such in the comment).

To understand the reason for specifying the "-force" option, consider the following  example where the deployment failed:


C:\NotBackedUp\Fabrikam\Builds\1.0.39.0\Portal\DeploymentFiles\Scripts&gt;<kbd>"Deploy Solutions.cmd"</kbd>

<samp>23:46 - Deploying solution: Fabrikam.Portal.StsAdm.Commands...<br><br>Deploying Fabrikam.Portal.StsAdm.Commands...<br>
<br>
Timer job successfully created.<br>
<br>
Executing .<br>Operation completed successfully.<br>
<br>
Done<br>23:46 - Deploying solution: Fabrikam.Portal.Web...<br>Deploying Fabrikam.Portal.Web on <a href="http://fabrikam-test/">http://fabrikam-test</a>...<br>
<br>
Timer job successfully created.<br>
<br>Executing .<br>Executing solution-deployment-fabrikam.portal.web.wsp-0.<br>The solution-deployment-fabrikam.portal.web.wsp-0 job completed successfully, but could not be properly cleaned up. This job may execute again on this server.<br>Operation completed successfully.</samp>


Note that the top-level Deploy Solutions.cmd script simply calls each Deploy  Solution.cmd script for the various WSPs. For this particular project, we have a  number of custom StsAdm.exe commands, in addition to a "Portal.Web" solution that  contains all of our various features.

You might be wondering why this failed, since the output clearly states "Operation  completed successfully" for both WSPs.

Browsing to the **Solution Management **page in SharePoint Central  Administration, the **Status **column for **fabrikam.portal.web.wsp**showed **Error**. Clicking the link to view more details showed  an error stating that the solution is already deployed. Well, duh, that's because  the timer job that was created to deploy fabrikam.portal.web.wsp ran twice!

If the timer job had been "properly cleaned up" then everything would be fine.

Note that this problem doesn't occur each and every time the solutions are deployed.  Unfortunately, it's one of those sporadic problems that tend to cause much heartburn  for anyone who works in the world of software (think deadlocks in SQL Server or  race conditions in a multithreaded application).

Also note that the specified WSP was actually deployed, which means the features  can be subsequently activated (i.e. the second "A" in the "DR.DADA" process). However,  I never like to leave the **Solution Management **page showing any  items with **Error **status (even if the error is benign, like this  scenario).

Therefore, whenever I encounter this error, I simply set the FORCE\_OPTION environment  variable and then redeploy the solutions:


C:\NotBackedUp\Fabrikam\Builds\1.0.39.0\Portal\DeploymentFiles\Scripts&gt;<kbd>set FORCE_OPTION=-force</kbd>  
C:\NotBackedUp\Fabrikam\Builds\1.0.39.0\Portal\DeploymentFiles\Scripts&gt;<kbd>"Retract Solutions.cmd"</kbd>  
...  
C:\NotBackedUp\Fabrikam\Builds\1.0.39.0\Portal\DeploymentFiles\Scripts&gt;<kbd>"Deploy Solutions.cmd"</kbd>  
...


Fortunately, when the "-force" option is specified, even if the deployment timer  job isn't properly cleaned up -- in other words, it isn't deleted -- no error occurs  when the solution is deployed the second time, and the **Status **column  on the **Solution Management **page shows **Deployed**.

If you find this problem occurs frequently in your environment, you might consider  always specifying the "-force" option when deploying SharePoint solutions (which  is what we recently did on my latest project).

