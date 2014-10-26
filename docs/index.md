
## Introduction to iViewer JavaScript

* [Introduction to iViewer JavaScript development](intro.md)
* [iViewer JavaScript Module Development Guidelines](modules.md)
* [Javascript startup mechanism, lifetime and execution context](startup.md)
* [iViewer Remote Monitor and JavaScript debugger](debug.md)
* [Javascript language documentation and resources](resources.md)
* [FAQ and How-Tos](howto.md)


## iViewer JavaScript API reference

The iViewer Javascript API provides a number of functions you can use to perform common tasks, interact with the GUI and with external systems.

* [API changes history](changes.md)
* [The CF object and its exposed variables](globals.md)
* [GUI API](gui.md)
    * _Events_
        * [CF.PreloadingCompleteEvent](gui.md#cF.PreloadingCompleteEvent) _GUI caching completed_
        * [CF.JoinChangeEvent](gui.md#cF.JoinChangeEvent) _a join changed_
        * [CF.InputFieldEditedEvent](gui.md#cF.InputFieldEditedEvent) _an input field being edited was modified_
        * [CF.GUISuspendedEvent](gui.md#cF.GUISuspendedEvent) _the app was put to background_
        * [CF.GUIResumedEvent](gui.md#cF.GUIResumedEvent) _the app came back to foreground_
        * [CF.ObjectPressedEvent](gui.md#cF.ObjectPressedEvent) _user pressed a button or slider_
        * [CF.ObjectDraggedEvent](gui.md#cF.ObjectDraggedEvent) _user is dragging a slider_
        * [CF.ObjectReleasedEvent](gui.md#cF.ObjectReleasedEvent) _user lifted finger from a button or slider_
        * [CF.KeyboardUpEvent](gui.md#cF.KeyboardUpEvent) _the software keyboard popped up_
        * [CF.KeyboardDownEvent](gui.md#cF.KeyboardDownEvent) _the software keyboard slided down_
    * _Functions_
        * [CF.getJoin()](gui.md#cF.getJoin) _get value and tokens of a join_
        * [CF.getJoins()](gui.md#cF.getJoins) _get value and tokens of multiple joins_
        * [CF.setJoin()](gui.md#cF.setJoin) _set value of a join_
        * [CF.setJoins()](gui.md#cF.setJoins) _set value and/or tokens of multiple joins_
        * [CF.setToken()](gui.md#cF.setToken) _set a join token_
        * [CF.getProperties()](gui.md#cF.getProperties) _get the visual properties of a join_
        * [CF.setProperties()](gui.md#cF.setProperties) _set and animate visual properties changes_
        * [CF.getGuiDescription()](gui.md#cF.getGuiDescription) _obtain a summary of all GUI objects_
* [Lists API](lists.md)
    * _Events_
        * [CF.ListWillStartScrollingEvent](lists.md#cF.ListWillStartScrollingEvent) _list will start scrolling_
        * [CF.ListDidScrollEvent](lists.md#cF.ListDidScrollEvent) _list is scrolling, first visible item just changed_
        * [CF.ListDidEndScrollingEvent](lists.md#cF.ListDidEndScrollingEvent) _list did end scrolling_
    * _Functions_
        * [CF.listAdd()](lists.md#cF.listAdd) _add or insert items in a list_
        * [CF.listUpdate()](lists.md#cF.listUpdate) _modify items in a list_
        * [CF.listRemove()](lists.md#cF.listRemove) _remove items from a list_
        * [CF.listScroll()](lists.md#cF.listScroll) _programatically scroll lists_
        * [CF.listInfo()](lists.md#cF.listInfo) _get info about a list_
        * [CF.listContents()](lists.md#cF.listContents) _get partial or complete list contents_
* [Display API](display.md)
    * _Events_
        * [CF.OrientationChangeEvent](display.md#cF.OrientationChangeEvent) _device changed orientation_
        * [CF.PageFlipEvent](display.md#cF.PageFlipEvent) _current page changed_
    * _Functions_
        * [CF.flipToPage()](display.md#cF.flipToPage) _flip to another page_
* [Network API](net.md)
    * _Events_
        * [CF.NetworkStatusChangeEvent](net.md#cF.NetworkStatusChangeEvent) _network went up or down_
        * [CF.ConnectionStatusChangeEvent](net.md#cF.ConnectionStatusChangeEvent) _a system connected or disconnected_
        * [CF.FeedbackMatchedEvent](net.md#cF.FeedbackMatchedEvent) _a feedback item was matched_
    * _Functions_
        * [CF.setSystemProperties()](net.md#cF.setSystemProperties) _change a system's settings_
        * [CF.send()](net.md#cF.send) _send data to a system_
        * [CF.runCommand()](net.md#cF.runCommand) _run a command_
        * [CF.runMacro()](net.md#cF.runMacro) _run a macro_
        * [CF.stopMacro()](net.md#cF.stopMacro) _stop one or all macros_
        * [CF.request()](net.md#cF.request) _perform HTTP/HTTPS requests_
        * [CF.startLookup()](net.md#cF.startLookup) _start a permanent Bonjour (mDNS/ZeroConf) lookup_
        * [CF.stopLookup()](net.md#cF.stopLookup) _stop a Bonjour lookup_
        * [CF.startPublishing()](net.md#cF.startPublishing) _start publishing a Bonjour service on the network_
        * [CF.stopPublishing()](net.md#cF.stopPublishing) _stop publishing a Bonjour service on the network_
* [Video API](video.md)
    * _Events_
        * [CF.MovieInfoReceivedEvent](video.md#cF.MovieInfoReceivedEvent) _new information about movie is available_
        * [CF.MovieLoadStateChangedEvent](video.md#cF.MovieLoadStateChangedEvent) _video buffering started, partially done or completed_
        * [CF.MoviePlaybackStateChangedEvent](video.md#cF.MoviePlaybackStateChangedEvent) _playback started, stopped or paused_
* [Sensors API](sensors.md)
	* _Functions_
		* [CF.startMonitoring()](sensors.md#cF.startMonitoring)
		* [CF.stopMonitoring()](sensors.md#cF.stopMonitoring)
* [Utilities API](util.md)
    * _Events_
        * [CF.DevicePropertyChangedEvent](util.md#cF.DevicePropertyChangedEvent) _a device property changed_
    * _Functions_
        * [CF.log()](util.md#cF.log) _debug function to log to monitor_
        * [CF.logObject()](util.md#cF.logObject) _debug function to dump objects_
        * [CF.crc()](util.md#cF.crc) _compute CRCs_
        * [CF.hash()](util.md#cF.hash) _compute hashes_
        * [CF.openURL()](util.md#cF.openURL) _open URLs in web browser, open other applications_
        * [CF.loadGUI()](util.md#cF.loadGUI) _load or reload a GUI with settings customization options_
        * [CF.loadAsset()](util.md#cF.loadAsset) _load asset files directly from JavaScript_
        * [CF.setDeviceProperty()](util.md#cF.setDeviceProperty) _set a device property, (i.e. screen brightness)_
