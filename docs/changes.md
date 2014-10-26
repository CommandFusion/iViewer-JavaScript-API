## iViewer JavaScript API change history

This page lists changes and additions to the iViewer JavaScript API itself.

#### iViewer 4.0.xxx

* Added support for `showPreloadStatus` boolean in [CF.loadGUI](util.hmlt#cF.loadGUI).
* Added new [CF.networkType](net.md#cF.networkType) property.
* Added new [CF.networkSSID](net.md#cF.networkSSID) property.

#### iViewer 4.0.242

* Added new [CF.device.displayDensity](globals.md#cF.device) property to allow detection of Retina displays
* Added new [CF.loadGUI()](util.md#cF.loadGUI) API
* [CF.watch()](gui.md#cF.watch) now logs meaningful information when incorrect parameters are used.
* Current JS context for gesture JavaScript now includes `join`, `tokens`, `list` and `listindex` local variables to identify the object that is configured with the gesture.

#### iViewer 4.0.218

* Added new [sensors (gyroscope, accelerometer)](sensors.md) support.
* Added new [CF.device.hasSensors](globals.md#cF.device) information about sensors that are supported
* Added new [CF.guiURL](gui.md#cF.guiURL) variable holding the original download URL of the GUI being run.

#### iViewer v4.0.210

* Added new [CF.ListWillStartScrollingEvent](lists.md#cF.ListWillStartScrollingEvent) event.
* Added new [CF.ListDidScrollEvent](lists.md#cF.ListDidScrollEvent) event.
* Added new [CF.ListDidEndScrollingEvent](lists.md#cF.ListDidEndScrollingEvent) event.
* Added new [CF.DevicePropertyChangedEvent](util.md#cF.DevicePropertyChangedEvent) event.
* Added new [CF.setDeviceProperty()](util.md#cF.setDeviceProperty) API.
* Added new [CF.device.screenBrightness](globals.md#cF.device) property to `CF.device`.
* Added new [CF.device.soundOutputVolume](globals.md#cF.device) property to `CF.device`.
* Added new [CF.device.batteryLevel](globals.md#cF.device) property to `CF.device`.
* Added new [CF.device.batteryChargeStatus](globals.md#cF.device) property to `CF.device`.

#### iViewer v4.0.6

* Added new `enabled` field in each system's description in the [CF.systems](net.md#cF.systems) global variable
* Added new [CF.setSystemProperties()](net.md#cF.setSystemProperties) API to allow stopping, starting and changing network address and ports of systems defined in the GUI.
* Added support for controlling GUI objects rotations in 3D. The new `xrotation`, `yrotation` and `zrotation` properties are returned by [CF.getProperties()](gui.md#cF.getProperties) and can be modified and animated using [CF.setProperties()](gui.md#cF.setProperties).
* [CF.setJoins()](gui.md#cF.setJoins) can now modify a join's GUI object properties.
* Added support for using any subpage as a list item's representation in [CF.listAdd()](lists.md#cF.listAdd) by setting an item's `subpage` property to the name of the subpage to use. [CF.listContents()](lists.md#cF.listContents) has also been updated to return this information.
* Added support for customizing the properties of GUI elements in list items through support for passing a `properties` object in the data for [CF.listAdd()](lists.md#cF.listAdd).
* Added new position specification `CF.VisiblePosition` to [CF.listScroll()](lists.md#cF.listScroll).
* Added new [CF.PreloadingCompleteEvent](gui.md#cF.PreloadingCompleteEvent), [CF.KeyboardUpEvent](gui.md#cF.KeyboardUpEvent) and [CF.KeyboardDownEvent](gui.md#cF.KeyboardDownEvent) events.
* Fixed issues in the JavaScript API where in some cases, strings containing international characters or 0xFF byte would not be correctly transmitted through [CF.send()](net.md#cF.send).
* [CF.setJoin()](gui.md#cF.setJoin) and [CF.setJoins()](gui.md#cF.setJoins) now always update the main control system when a join value changes.
* [CF.GUISuspendedEvent](gui.md#cF.GUISuspendedEvent) will now be correctly received at the time the application goes to background. Any CF call made by this callback will be executed immediately.

#### iViewer v4.0 (public release v4.0.5)

Initial availability of the iViewer JavaScript API.
