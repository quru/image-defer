# image-defer changelog

### v0.3 Defences against bad options

Bug fixes:

* Make `ImageDefer.options` a read-only property, to prevent it from being accidentally
  overwritten (e.g. when setting options after image-defer has initialised).
  This does not prevent the individual options themselves from being changed.
* Prevent `ImageDefer.options.scrollingStopMillis` from being 0, undefined, or too low
* Do not unload images if `ImageDefer.options.maxLoaded` is 0 or undefined

### v0.2 Missing option

Bug fix:

* It was not possible to override `scrollingSkipRate` from the options.

### v0.1 Development version

This is the first usable version, and has had limited testing.
