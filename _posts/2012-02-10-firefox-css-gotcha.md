---
title: Firefox CSS gotcha
redirect_from:
  - /posts/firefox-css-gotcha/
---

Last week I coded my own version of the New York Times' article slider panel in JS. Recently I have been using Chrome as my development browser; I switched over from Firefox a few months ago. After verifying that the slider worked correctly in Chrome, I was surprised to find that it did not work at all in Firefox. In fact, Firefox was not loading any of my external CSS or JS pages. Here is a look at the "head" element of my demo HTML page:

```
<head>
    <link rel="stylesheet" href="styles.css" />
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="chrome=1,IE=9" />
    <meta name="viewport"/>
    <script type="text/javascript" src="jq.js"></script> 
</head>
```


Spot the problem? That's right, the first line does not have a "type" attribute. It should look like this:



```
<link rel="stylesheet" href="styles.css" type="text/css" />
```


Firefox 9.0 will not load external CSS or JS pages without the "type" attribute! Chrome 17.0, Safari 5.1, and IE 8 do not need the type attribute, so be sure to test in Firefox if you develop in any of these.