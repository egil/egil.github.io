---
layout: post
title: Get background image to resize with browser window using CSS and jQuery
created: 1319268101
---
Using a photo as a background for a web page can look pretty good, but making that photo fill out the entire browser window no matter its size and proportions is not straight forward, if you also want to keep the photo in its original aspect ratio. Not clear what I mean -- check out the [demo page](/sites/default/files/node/29/auto-resize-background-image-demo.html) to see the effect in action (try resizing it).

Some of the new [CSS3 tricks come pretty close]( http://www.css3.com/css3-background-size/), but they still seem to lack the ability to let the user expand the browser window in any direction and also keep the photos aspect ratio. If we also want a solution that works with older browsers (e.g. IE7/IE8), we need to go a different route.

## jQuery to the rescue

First let's add the HTML to our page that will display the photo. The HTML can be placed anywhere inside your pages body tags, but I recommend putting it at the very bottom to keep it out of the way. Make sure to update the `src` attribute to point to the photo you want as your background.

```html
<div id="bgimgwrapper">
    <img alt="" src="photo.jpg" id="bgimg" style="width: 100%;" />
</div>
```

Next we need to add the following CSS to our stylesheet. This code will fix the `bgimg` to the page (`position: fixed;`), and make sure it is positioned behind everything else on the page (`z-index: -1;`).

```css
#bgimgwrapper {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  overflow: hidden;
}

#bgimg { 
  display: block; 
  -ms-interpolation-mode: bicubic; 
}
```

Last we need the following JavaScript added to our page. The script makes sure the photo resizes itself properly when the browser window is resized, while keeping its original aspect ratio intact. To make the script run a little faster, we only update the `#bgimg` DOM element when we absolutely have to, which is when we switch between having the image expand in height or width.

```js
(function($){
  var pr = 0, bgWidth = true, bgResize;

  bgResize = function(){
    var wr = $(window).width() / $(window).height();

    if(wr > pr && !bgWidth) {
      $('#bgimg').css('width', '100%').css('height', '');
      bgWidth = true;
    } else if(wr < pr && bgWidth) {
      $('#bgimg').css('height', '100%').css('width', '');
      bgWidth = false;
    }
  };

  $(document).ready(function() {
    var img = $('#bgimg');
    if(img.height()) {
      pr = $('#bgimg').width() / $('#bgimg').height();
      bgResize();
    } else {
      img.load(function(){
        pr = $('#bgimg').width() / $('#bgimg').height();
        bgResize();
      });
    }

    $(window).resize(bgResize);
    prettyPrint();
  });
})(jQuery);
```

If anybody has a way to improve on this, please post a comment. As far as I can see the code is pretty robust, but I have only tested in Chrome, Internet Explorer [7|8|9], Firefox 6, and Safari 5.1, so there might be issues I am not aware of.

Go to the [demo page](/sites/default/files/node/29/auto-resize-background-image-demo.html) check out the finished result.
