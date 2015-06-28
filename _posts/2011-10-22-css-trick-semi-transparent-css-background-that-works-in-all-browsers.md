---
layout: post
title: 'CSS trick: Semi transparent CSS background that works in all browsers'
created: 1319256718
---
This simple CSS class will create a semi transparent background, letting whatever is underneath the element shine through. It works across all browsers, that means Internet Explorer 6.0 as well. The newest browsers will use the `background:rgba(0,0,0,.75);` setting, the rest (IE6-IE8) will use the gradiant filter to achieve the same effect.

```css
.black {
    background:rgb(0,0,0);
    background:rgba(0,0,0,.75);
    *background:transparent;
    *filter:progid:DXImageTransform.Microsoft.gradient(startColorStr=#BF000000,endColorStr=#BF000000);
    *-ms-filter:"progid:DXImageTransform.Microsoft.gradient(startColorstr=#BF000000,endColorstr=#BF000000)";
}
```

If you want to change the transparency level, you have to adjust both the RGBA values and the color values in the gradient filters. For example will `background:rgba(0,0,0,.4);` and `#6F000000` make elements more transparent.

For more opacity/rgba tricks, take a look at the article [CSS Opacity: A Comprehensive Reference](http://www.impressivewebs.com/css-opacity-reference/) over at [Impressive Webs](http://www.impressivewebs.com). The code snippet above was originally found on [pastie.org](http://pastie.org/866475), all credit to the person who posted it there.
