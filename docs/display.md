
# Display API

The Display API allows a script to get the current page name and orientation, as well as being notified when either one changes.

## Constants and variables

The Display API defines two constants, `CF.PortraitOrientation` and `CF.LandscapeOrientation` which you can use to test the value of `CF.currentOrientation`. The `CF` namespace also holds two variables that are updated every time a change occurs:

* `CF.currentPage` is always up to date with current page name
* `CF.currentOrientation` is always up to date with the current display orientation


## Events

#### CF.OrientationChangeEvent

This event is fired when the display orientation successfully changes (that is, when the device rotates and the current page supports the new orientation). Your watcher callback function gets called with two parameters: the current page name, and the new orientation. Use `CF.watch()` to watch orientation changes. Optional last `true` parameter will fire your callback once with the current display orientation:

    function onOrientationChange(pageName, newOrientation) {
        CF.log("Orientation of page " + pageName + " changed to: " + newOrientation);
        if (newOrientation == CF.LandscapeOrientation) {
            // Do something when switching to landscape
        } else {
            // Do something when switching to portrait
        }
     }

    CF.userMain = function() {
        // Start watching orientation changes, asking for initial fire with the current page name
        // and orientation
        CF.watch(CF.OrientationChangeEvent, onOrientationChange, true);
    };


To stop watching page orientation changes, use `CF.unwatch()`:

    // Remove all the orientation change watchers
    CF.unwatch(CF.OrientationChangeEvent);


#### CF.PageFlipEvent

This even is fired when a page flip occurs. Your callback receives the previous page name, new page name and new page orientation. iViewer will try to flip to the new page using the current screen orientation, but if the new page doesn't support the current orientation then the other orientation will be used. Optional last `true` parameter will fire your callback once with the current page name, previous page name will be `null`.

    function onPageFlip(from, to, orientation) {
        if (from == null)
            CF.log("PageFlipEvent armed, current page is " + to + " with orientation " + orientation);
        else
            CF.log("Page changed from " + from + " to " + to + " with orientation " + orientation);
    }

    CF.userMain = function() {
        // Start watching orientation changes, asking for initial fire with the current page name
        // and orientation
        CF.watch(CF.PageFlipEvent, onPageFlip, true);
    };

To stop watching page flips, use `CF.unwatch()`:

    // Remove all the page flip watchers
    CF.unwatch(CF.PageFlipEvent);


## Functions

#### CF.flipToPage(pageName)

You can manually flip to the page you want by calling this function. Pass the name of the page you want to go to. Note that this _will_ trigger a `CF.PageFlipEvent`.
Example:

    // Flip to a page named "Artists"
    CF.flipToPage("Artists");

