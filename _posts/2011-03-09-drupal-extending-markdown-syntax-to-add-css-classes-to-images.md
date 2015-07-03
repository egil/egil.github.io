---
layout: post
title: 'Drupal: Extending Markdown syntax to add CSS classes to images'
created: 1299657600
redirect_from:
  - /2011/03/09/drupal-extending-markdown-syntax-add-css-classes-images
---
**Update June, 2013:** As pointed out by [Don Morris in the comments](http://egilhansen.com/2011/03/09/drupal-extending-markdown-syntax-add-css-classes-images#comment-920241107), PHP Markdown Extra now supports classes, making the below trick obsolete for this purpose.

---------

If you are using input filters such as the [Markdown filter module](http://drupal.org/project/markdown), you may run in to a situation where the script behind the module does not provide you with all the formatting options you want.

This was the case for me with a Drupal site I am currently building, I wanted to allow the editors of the site to specify the position of the image, i.e. float left, float right, or centered, but this is not supported by the otherwise [excellent Markdown syntax](http://daringfireball.net/projects/markdown/syntax), so I decided to use the [Custom filter module](http://drupal.org/project/customfilter) to solve my problem.

Another option is to hack the Markdown module or Markdown itself, but neither is a good option as newer versions of either could break the hack. The solution with the Customer filter module does not do this, since it just adds a new filter that can be applied after the Markdown filter. Straight from the Custom filter page:

> Instead of creating a new filter module for every filter you need, you can use this module [Custom filter] for tasks like creating your own tags, replacing token values and all changes you need in the text.

The syntax I wanted to create looks like this: `![Alt text](/path/to/img.jpg){css classes}`

What differs from the original Markdown image syntax is the `{css classes}` at the end. The idea is to basically add what is between the curly brackets as css classes to the img tag, so that end result will be:

~~~html
![Alt text](/path/to/img.jpg){css classes} becomes
<img src="/path/to/img.jpg" alt="Alt text" class="css classes" />
~~~

With this in place, it is easy to control left and right floating of an image by having the user write `{left}` or `{right}` and the have the appropriate styling in your style sheet.

How to add custom css classes to the generated img tag:

1. First install and configure the Markdown module as you normally would.
2. Then install the Custom filters module.
3. Go to admin/settings/customfilter and create a new Filter Set.
4. Add a new Replacement Rule to the just created filter with the following data:  
**Name**: whatever you like  
**Description**: whatever you like  
**Pattern**: `/(<img.*)\/>\{([A-Za-z0-9-_\s]*)\}/`  
**Replacement text**: `${1}class="${2}" />`
5. Go to your Markdown text filter (input filter in Drupal 6) and select you newly created filter set and make sure that it is listed below Markdown in the filter processing order list.

For more information about creating filters with the Custom filter module, see the [Getting Started section](http://drupal.org/node/210551) and the [Creating filters section](http://drupal.org/node/210553) in the documentation for the Custom filter module.
