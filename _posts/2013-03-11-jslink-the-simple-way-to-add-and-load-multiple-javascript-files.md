---
layout: post
title: 'JSLink: The simple way to add and load multiple JavaScript files'
created: 1363014256
redirect_from:
  - /2013/03/11/jslink-simple-way-add-and-load-multiple-javascript-files/
---
It can sometimes be useful to specify multiple JavaScript files to use when rendering e.g. a field in SharePoint 2013. It could be that you have a JavaScript file with some shared code that you use in all your custom rendering, or you use a library such as jQuery. In either case, you need all the scripts loaded before you can start rendering.

<!--break-->

It turns out it is easy to tell SharePoint to download multiple JavaScript files when rendering e.g. a field. Just separate them with the pipe character, i.e. `|`.

For example, to load `~sitecollection/_catalogs/masterpage/jQuery.js` and `~sitecollection/_catalogs/masterpage/my-custom-field.js`, set the JSLink property of your target to:

`JSLink="~sitecollection/_catalogs/masterpage/jQuery.js|~sitecollection/_catalogs/masterpage/my-custom-field.js"`

Do not worry about SharePoint re-downloading the same script repeatedly if you use it to render multiple elements, SharePoint will not download a script more than one time (e.g. jQuery).
