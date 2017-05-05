# image-defer.js

image-defer is a simple, dependency-free JavaScript library for lazily loading images on a web page. In a nutshell, images that you choose to defer are not loaded until the user can see them, while images that remain "off the page" are never loaded.

Lazily loading images has these up-sides:

* Reduced server load, server bandwidth, and client-side bandwidth, by not loading images that the user never sees
* Flatter server load, by not requesting all images at once
* Faster loading web pages
* Reduced (and also capped) client-side memory usage

and these down-sides:

* Requires a little extra CPU power when the user scrolls the web page
* The user may have to wait a short time for images to load when they come into view
* You need to define your image sizes in advance (to create place-holders so that the page does not jump around as images load)
* The user needs to have JavaScript enabled to view the images

This library was developed for, and is bundled with, the [Quru Image Server](https://github.com/quru/qis), but functions perfectly well on its own, for use with either static or dynamic images.

Unlike some other "modern" lazy loading libraries, this one does not require babel, webpack, rollup, npm, node, grunt, gulp, or any of that business.

### Live demo

TODO Publish the test page on github.io

TODO Reload should be smooth using browser cache

TODO Ack the icons - The loading icons are modified from the Essential Collection pack, by madebyoliver at www.flaticon.com.

### Running

TODO include script, can be async or defer

TODO Define fixed image sizes in css, data-defer-src in html

TODO Explain Y axis only loading - works fine with horizontal carousels, may load off-screen images on mobile but horizontal scrolling on mobile is bad for usability anyway.

TODO Describe events - requested event not called when images are swapped in from browser cache - loaded and unloaded should be

TODO Call reset if the page layout changes

TODO Describe image strips

### Options

TODO Describe options

TODO Limit may be exceeded - briefly when loading a new strip - or permanently if more images are visible than the limit

TODO Explain options definition if async

### Browser support

Based on [MDN](https://developer.mozilla.org/)'s browser support tables,
the library should in theory support these browsers (or newer):

* IE 9 or Edge
* Firefox 4
* Chrome 7
* Safari 5.1
