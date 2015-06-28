---
layout: post
title: SharePoint Foundation 2010 on Windows 7/8 – missing config.xml file
created: 1373358114
---
To install SharePoint Foundation 2010 on Windows 7/8, you need to modify the config.xml file. However, after extracting the setup files, the config.xml file was nowhere to be seen in the Setup folder. To get the installation going, I had to create the file myself. Here is the content that goes into it:

~~~XML
<Configuration>
   <Package Id="sts">
      <Setting Id="SETUPTYPE" Value="CLEAN_INSTALL"/>
   </Package>
   <DATADIR Value="%CommonProgramFiles%\Microsoft Shared\Web Server Extensions\14\Data"/>
   <Logging Type="verbose" Path="%temp%" Template="Microsoft SharePoint Foundation 2010 Setup *.log"/>
   <Setting Id="UsingUIInstallMode" Value="1"/>
   <Setting Id="SETUP_REBOOT" Value="Never"/>
   <Setting Id="AllowWindowsClientInstall" Value="True"/>
</Configuration>
~~~
And to force the setup.exe to use it, we need to apply the /config switch when starting setup.exe, i.e.: `setup.exe /config Setup\config.xml`

Other useful links for installing SharePoint Foundation 2010 on Windows 8:

- [SharePoint 2010 Management Shell does not load with Windows PowerShell 3.0](http://support.microsoft.com/kb/2796733) - because SharePoint PowerShell does not work out of the box.
- [Develoq.net: Installing Sharepoint Foundation 2010 on Windows 7](http://develoq.net/2010/installing-sharepoint-foundation-2010-on-windows-7/) - how to install on Farm mode on Windows 7/8, to avoid getting another SQL server instance.
- [Installing SharePoint 2010 on Windows 8](http://johnlivingstontech.blogspot.dk/2011/09/installing-sharepoint-2010-on-windows-8.html) - Hints about getting installation to work on Windows 8, also good tips in comments.

Hope this helps.
