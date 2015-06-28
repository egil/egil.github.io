---
layout: post
title: 'SharePoint 2013: The server was unable to save the form at this time'
created: 1376474695
---
This morning our freshly installed SharePoint Server 2013 server was refusing to let me save a simple form with a Lookup column to another list. Using the standard insert/edit form, the error message read:

===
![](sharepoint-error-message-form.gif)
===[Error message from insert/edit form: *The server was unable to save the form at this time. Please try again.*]

When using the quick edit mode, the error message was:

===
![](sharepoint-error-message.gif)
===[Error message in quick edit mode: *This service isn't available right now. Click here to try again.*]

The fix turned out to be a `iisreset` on the SharePoint server.

**Update:** It looks like the culprit is the Security Token Service, more info over at [Sharepoint2013.me](http://www.sharepoint2013.me/Blog/Post/83/-The-Security-Token-Service-is-not-available--error-in-sharepoint-2013).

I hope this helps somebody else save a few hours of head scratching.
