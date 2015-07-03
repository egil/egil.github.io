---
layout: post
title: SharePoint’s formula syntax can change depending on a sites regional setting
created: 1360844835
redirect_from:
  - /2013/02/14/sharepoint-s-formula-syntax-can-change-depending-sites-regional-setting/
---
Sometimes very simple formulas like `=IF([Column1]<=[Column2], "OK", "Not OK")` can result in the unhelpful <cite>"The formula contains a syntax error or is not supported"</cite> error message, even though the formula in the example is perfectly valid according to the [documentation]( http://office.microsoft.com/en-001/windows-sharepoint-services-help/examples-of-common-formulas-HA001160947.aspx), it still wont accept it.

<!--break-->

## The solution is in the comma ##
It turns out that a SharePoint sites regional setting changes the meaning of the comma (,) in the formula syntax. The solution is simple (if you know it): **Just use a semi colon instead (;)**.

The example above should then be changed to `=IF([Column1]<=[Column2]; "OK"; "Not OK")` and everything is OK.
