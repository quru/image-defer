# image-defer.js

image-defer is a simple, fast, dependency-free JavaScript library for lazily loading images on a web page. In a nutshell, images that you choose to defer are not loaded until the user can see them, while images that remain "off the page" are never loaded.

Lazily loading images has these up-sides:

* Reduced server load, server bandwidth, and client-side bandwidth, by not loading images that the user never sees
* Flatter server load, by not requesting all images at once
* Faster loading web pages
* Reduced client-side memory usage - initially at least
* Capped client-side memory usage - with `ImageDefer.options.maxLoaded`

and these down-sides:

* Requires a little extra CPU power when the user scrolls the web page
* You need to know the image dimensions in advance (use fixed sizes so that the page does not jump around as images load)
* The user may have to wait a short time for images to load when they come into view
* The user needs to have JavaScript enabled to view the images

This library was developed for, and is bundled with, the [Quru Image Server](https://github.com/quru/qis), but functions perfectly well on its own, for use with either static or dynamic images.

Unlike some other "modern" lazy loading libraries, this one does not require babel, webpack, rollup, npm, node, grunt, gulp, or any of that business.

## Live demo

You can try out image-defer on the live demo page at [https://quru.github.io/image-defer/](https://quru.github.io/image-defer/).

View the source if you want to see the plumbing, and clear your browser cache before refreshing the page if you want the images to load again remotely rather than coming from the local browser cache.

_Acknowledgements_: the placeholder and loading icons are modified from the Essential Collection pack, by _madebyoliver_ at [flaticon.com](http://flaticon.com/).

## How to use

### Adding the script

First include the image-defer script in your web page HTML:

    <script type="text/javascript" src="//your.server/path/to/image-defer.min.js" defer></script>

The script has no dependencies so you can use the `async` script tag attribute if you wish.
It does not need to run until the DOM is complete, so you can also use the `defer` script tag
attribute (unrelated to image-defer), as in the example above. See
[MDN](https://developer.mozilla.org/en/docs/Web/HTML/Element/script) for more information about
these options.

The library creates a single global object called `ImageDefer`, so it should be safe to use
alongside or to combine with any other JavaScript library that does not try to create the same.

### Defining the images

In your HTML, define images using the normal `img` tag, but have the initial `src` attribute point
to a common placeholder image, and define a new `data-defer-src` attribute to contain the proper
image URL:

    <img src="placeholder.jpg" data-defer-src="//your.server/path/to/final-image.jpg">

Any images that do not have a `data-defer-src` attribute will simply be ignored by image-defer.

The placeholder image should be the same for all your lazily loaded images, so that the browser
only loads one copy to display in all of them.

You need to use CSS, or `width` and `height` attributes, to define fixed sizes for all the lazy
images. This is so that the browser reserves the correct space for them in the page ahead of time.
Failure to do this causes 2 problems:

1) A really bad user experience, as the browser has to layout the web page again every time
   an image arrives, causing everything to jump around  
2) Image-defer, which previously recorded the locations of all the images, will not know that
   they have moved, and will start to lazy load from the wrong positions

If you don't care about (1) and you want to fix (2) then you can call `ImageDefer.reset()`
once the new page layout has completed.

For an example of CSS that defines a reusable placeholder independent of the image size,
see the source of the [demo page](docs/index.html). The placeholder is created with the
`spacer.png` file and the `defer` CSS class, and works for any image size. The `thumbnail`
CSS class defines the final image size, so you could easily define different classes for
different image sizes.

### Running

The library initialises and runs automatically as soon as the web page has loaded, during the
page's `DOMContentLoaded` event. If it works and you are happy with the defaults then there is
nothing else to do, but if not then read on.

In order to remain fast even for very large web pages, the library breaks down the web page into
horizontal bands, and then deals only with the currently visible bands. What this means in
practice is that image-defer only supports lazy loading together with vertical scrolling. This
covers most use cases. If your page scrolls horizontally, then images will be loaded regardless
of whether they are visible or horizontally off the page. This should be compatible with most
sideways-animating image slider (carousel) controls. This design is also why you might notice
horizontal groups of images loading and unloading together.

The library detects and handles window resizing and orientation changes (on mobile) automatically.
If the page layout changes dynamically in another way, you might need to tell image-defer to update
its record of those horizontal bands and what is in them. To do this, call `ImageDefer.reset()`
whenever the page layout has changed (vertically).

If you add or remove an image dynamically at runtime, you can tell image-defer to lazy load it,
or stop lazy loading it, with the `ImageDefer.addImage()` and `ImageDefer.removeImage()` functions
(see _Available functions_ below). In fact you must call `removeImage` before deleting an image
if you want it to be garbage collected, otherwise image-defer's reference to it will cause it to
be kept in memory.

If you have a very large number of images on your page and will be using the limit feature
(i.e. having more images than `ImageDefer.options.maxLoaded`), ensure that the HTTP caching
headers on your image server are set to encourage client-side caching. If the user scrolls
from the bottom of the page back up to the top, you want the unloaded images to be lazily
loaded back in again from the browser's cache rather than being re-fetched from the server. 
That said, the correct setting of your HTTP caching headers is important for every scenario.

### Runtime events

The library emits up to 3 events for each image it finds with a `data-defer-src` attribute:

* `onImageRequested` - called as soon as the URL in `data-defer-src` has been requested.
  This event is skipped if the URL loads immediately (if it was already cached by the browser).

* `onImageLoaded` - called as soon as the URL in `data-defer-src` has finished loading
  (from the remote server or from cache) and has been displayed.

* `onImageUnloaded` - called when the `ImageDefer.options.maxLoaded` limit has been exceeded,
  for images that have been "unloaded" by setting them back to the initial placeholder.

You can subscribe to these events by supplying callback functions in `ImageDefer.options`
(see _Available options_ below). Each callback requires the following prototype:

    function callback(img) {
        // img is the image DOM element being loaded or unloaded
    }

These events can be used to update your user interface with the loading state of each image.
For an example of how to do this, see the source of the [demo page](docs/index.html).

### Available options

You can define any or all of the following attributes in `ImageDefer.options`:

* `maxLoaded` (default 100) - the maximum number of images to lazy load before starting to unload images again.
  This prevents the memory use of the web page from becoming too high. Images will only be unloaded when they
  are off-screen, so if the number of visible images exceeds the limit then the limit will be breached.  
  :warning: A common browser bug currently affects the way that image-defer implements this function,
  causing the memory use of the page to continue growing in some browsers :cry: See issue #2 for more details.

* `onImageRequested` (default none) - a callback function for the _image requested_ event, see _Runtime events_ above.

* `onImageLoaded` (default none) - a callback function for the _image loaded_ event, see _Runtime events_ above.

* `onImageUnloaded` (default none) - a callback function for the _image unloaded_ event, see _Runtime events_ above.

* `scrollingStopMillis` (default 500) - the time to wait, in milliseconds, before considering that a scrolling
  event has completed. Visible images are lazy loaded when scrolling stops.

* `scrollingSkipRate` (default 0.8) - the speed of scrolling, in pixels/millisecond, above which images will
  not be lazy loaded. Reducing this value, e.g. to `0.2`, means that lazy loading will not take place while
  the user is scrolling quite slowly. Raising it, e.g. to `1.5`, means that images will still be lazy loaded
  even when the user is scrolling quickly.

You can define `ImageDefer.options` before or after including `image-defer.js` in the page, but if image-defer
loads first then your code needs to alter the existing options object rather than replacing it with a new one.

Setting options before image-defer has loaded:

    // Pre-define image-defer if it has not yet loaded
    window.ImageDefer = window.ImageDefer || {};
    
    ImageDefer.options = {
        maxLoaded: 50,
        scrollingSkipRate: 0.5
    };

Setting options after image-defer has loaded:

    ImageDefer.options.maxLoaded = 50;
    ImageDefer.options.scrollingSkipRate = 0.5;

### Available functions

Image-defer is intended to be fully automatic in the majority of cases, but there might be times
when you need to interact with it. The following functions are considered to be public:

* `ImageDefer.supportedBrowser()` - Returns a boolean for whether the browser environment appears to
  support image-defer. If this returns `false` then image-defer will not attempt to run automatically.

* `ImageDefer.reset()` - Re-scans the page for all `img` elements, and records the positions of all
  those with a `data-defer-src` attribute for lazy loading. You may need to call this if your images
  move (vertically) at some point after the page has finished loading.

* `ImageDefer.addImage(element)` - Tells image-defer to take on the lazy loading of an `img` element.
  You may need to call this if an image is dynamically added to the page. It takes no effect if the
  image does not have a `data-defer-src` attribute.

* `ImageDefer.removeImage(element, load)` - Tells image-defer to forget about an `img` element.
  This takes no effect if `addImage` was not called first. Optionally you can pass `true` for the
  second parameter to tell image-defer to eager load the image before it stops monitoring it.

* `ImageDefer.lazyLoadImages()` - Tells image-defer to load all currently visible images. You may need
  to call this if the scroll position of the page changes without having emitted the normal scroll events.

* `ImageDefer.trimImages()` - Tells image-defer to unload images, if possible, if it has loaded more
  than the `ImageDefer.options.maxLoaded` limit. Takes no effect if the limit has not been reached.

* `ImageDefer.imagesLoaded()` - Returns the number of images that are currently loading or loaded,
  i.e. loading or displaying the image specified in the `data-defer-src` attribute.

## Browser support

Based on [MDN](https://developer.mozilla.org/)'s browser support tables,
the library should in theory support these browsers (or newer):

* IE 9 or Edge
* Firefox 4
* Chrome 7
* Safari 5.1

You can use the live demo page to see if it works for a particular browser.
