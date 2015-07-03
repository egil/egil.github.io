---
layout: post
title: Remember the FeatureId when creating a custom ListInstance in SharePoint 2010
created: 1349456215
redirect_from:
  - /2012/10/05/remember-featureid-when-creating-custom-listinstance-sharepoint-2010
---
When creating a list instance manually in a SharePont 2010, it is important to remember to add the right `FeatureId` attribute to the [ListInstance]( http://msdn.microsoft.com/en-us/library/ms476062.aspx) element, if the `TemplateType` attribute points to a list template which is not in the same feature as the one you are creating the list instance in. If you forget, you will get some very vague error messages.

<!--break-->

I was creating a list instance with a custom schema attached and when I tried to deploy I got this error message:

> Error occurred in deployment step 'Activate Features': Invalid file name.
>
> The file name you specified could not be used.  It may be the name of an existing file or directory, or you may not have permission to access the file.

At first I thought it was because of the custom schema, so I removed it, but that just resulted in this even more vague error message:

> Error occurred in deployment step 'Activate Features': Cannot complete this action. 
>
> Please try again.

After digging around the [SharePoint 2010 documentation a bit]( http://msdn.microsoft.com/en-us/library/ms476062.aspx), I found out that I was missing the `FeatureId` attribute in my ListInstance declaration. If left out, SharePoint assumes that the list template exists in the same feature as the list instance, which was not the case, since I had the `TemplateType` set to 100 (Custom List). With `FeatureId="00bfea71-de22-43b2-a848-c05709900100"` everything started playing nice again.

The final `ListInstance` declaration including a `FeatureId` attribute matching the `TemplateType` attribute:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Elements xmlns="http://schemas.microsoft.com/sharepoint/">
  <ListInstance Title="Invoices"
                OnQuickLaunch="TRUE"
                TemplateType="100"
                FeatureId="00bfea71-de22-43b2-a848-c05709900100"
                Url="Lists/Invoices"
                Description="Invoices."
                VersioningEnabled="TRUE"
                DocumentTemplate=""
                CustomSchema="InvoicesListInstance/Schema.xml">
  </ListInstance>  
</Elements>
```

I found two pages on the net with list templates and their Feature Id's, hopefully this will help somebody else save a little hair.

- [List of feature ID, listTemplate (Vinit`s SharePoint Space)](http://blogs.technet.com/b/vinitt/archive/2009/11/04/list-of-feature-id-listtemplate.aspx)
- [SharePoint 2010 (Mostly) Complete List of List Templates (More Soma Please)](http://platinumdogs.me/2011/10/06/sharepoint-2010-list-list-templates/)
