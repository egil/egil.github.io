---
layout: post
title: 'CSS trick: Always show scrollbars'
created: 1175069795
redirect_from:
  - /2011/09/27/css-trick-always-show-scrollbars
---
**Update 2011:** Since Internet Explorer now always just display a vertical scrollbar, all that is needed to force a scrollbar in all other non-IE browsers is this:

```css
html {
    overflow-y: scroll;
}
```

---

I worked on a site a few weeks ago where the user wanted visible scrollbars no matter how much space the content takes up on the page. Internet Explorer is happy to oblige, just put in a `overflow: scroll;` in your CSS and your good to go. However, Mozilla Firefox wouldn't play ball. A bit of Googling revealed Mozillas own implementation:

```css
overflow-y: scroll;
```

So to sum up, add the following CSS code to your site if you want the scrollbars visible at all times:

```css
html {
     overflow: -moz-scrollbars-vertical;
     overflow: scroll;
}
```
