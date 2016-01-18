---
layout: post
title: Extra Tips for Debugging SharePoint Timer Jobs
created: 1373895392
redirect_from:
  - /2013/07/15/extra-tips-debugging-sharepoint-timer-jobs/
---
There is an excellent guide over on [MSDN on how to debug a SharePoint timer job]( http://msdn.microsoft.com/en-us/library/ff798310.aspx) from Visual Studio. However, it fails to mention that sometimes you need to restart the service that runs the timer jobs to allow debugger to set a breakpoint. In addition, sometimes a job will not show up in the job definitions list (Central Administration – Job Definitions), making it hard to manually start a job. The solution to this is to recycle IIS.

<!--break-->

The simplest way to incorporate this into a development workflow is to add it to these commands to the *Pre- and Post-deployment Command Line* steps in Visual Studio:

**Pre-deployment Command Line:**

~~~
net stop SPTimerV4
~~~

**Post-deployment Command Line:**

~~~
net start SPTimerV4
iisreset
~~~

Hope this helps.
