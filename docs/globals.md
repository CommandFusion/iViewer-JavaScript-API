# Global variables and constants

iViewer defines one global variable (`CF`) which itselfs holds a number of variables that you can use in your script. `CF` also contains a number of constants to be used with several of the iViewer APIs.

## CF variables summary

The Javascript interface to iViewer defines a single object: `CF`. All functions and variables exposed by iViewer exist in this object, so as to completely encapsulate the iViewer interface into a single object.

You are strongly advised not to change anything in the `CF` object. Variables are updated automatically by iViewer itself, any manual change you make to the documented variables will be overwritten. Moreover, any change you make to _undocumented_ variables may break the interface between JavaScript and iViewer.

The `CF` object exposes a number of variables:

* [CF.device](#cF.device) details about the device currently running your code
* [CF.currentPage](display.md#cF.currentPage) (_string_) the current page name
* [CF.currentOrientation](display.md#cF.currentOrientation) the current device orientation (`CF.PortraitOrientation` or `CF.LandscapeOrienation`)
* [CF.ipv4address](net.md#cF.ipv4address) (_string_) the device's IPv4 address
* [CF.ipv4netmask](net.md#cF.ipv4netmask) (_string_) the device's IPv4 subnet mask
* [CF.ipv6address](net.md#cF.ipv6address) (_string_) the device's IPv6 address
* [CF.ipv6netmask](net.md#cF.ipv6netmask) (_string_) the device's IPv6 subnet mask
* [CF.MACaddress](net.md#cF.MACaddress) (_string_) the device's network adapter MAC address
* [CF.networkType](net.md#cF.networkType) (_string_) the device's connection type
* [CF.networkSSID](net.md#cF.networkSSID) (_string_) the device's current connected network SSID when applicable
* [CF.systems](net.md#cF.systems) (_object_) a object listing all the external systems listed in your GUI
* [CF.guiURL](gui.md#cF.guiURL) (_string_) the original download URL (even when GUI has been cached) for the current GUI

### CF.device
Information about the current device if available in the `CF.device` object, which exposes the following properties:

* `platform` (_string_): the device OS platform (i.e. "iOS")
* `version` (_string_): the OS version (i.e. "4.3")
* `model` (_string_): the device model (i.e. "iPad")
* `uuid` (_string_): the device's unique ID string. On some iOS devices where the legacy unique identifier can still be retrieved, this property will contain either the legacy unique identifier (Apple's) or the iViewer-generated unique identifier, depending on against which one the license matched.
* `legacyUniqueIdentifier` (_string_): on iOS devices only, Apple's unique device identifier if it is available
* `uniqueIdentifier` (_string_): a unique identifier for this device, generated once by iViewer. On non-iOS devices, this is the same as the `uuid` property.
* `name` (_string_): the device name. For iOS devices, the is the name set in iTunes for this device.
* `screenBrightness` (_number_): the device's current screen brightness, a value between 0.0 and 1.0. Note that this property is always equal to 1.0 on iOS 4 as it is only supported on iOS 5 and later, and on Android platforms.
* `soundOutputVolume` (_number_): the device's sound output volume, a number between 0.0 (muted) and 1.0 (full volume)
* `batteryLevel` (_number_): the device battery charge level, a number between 0.0 (empty) and 1.0 (full)
* `batteryChargeStatus` (_enum_): the device charge status. See the [battery charge constants](util.md#battery_charge_constants) for possible values.
* `hasSensors` (_object_): an object with boolean values for each sensor (`CF.Gyroscope`, `CF.Accelerometer`) indicating whether the sensor is supported on the current device. See the [Sensors](sensors.md) API introduction for code that detects sensor support.
* `displayDensity` (_number_): indicates the density of the display compared to a regular display. This allows detection of Retina displays on iOS: 1.0 is standard density, 2.0 is a retina display. On Android, this value is always 1.0 for current versions of the OS and line of devices.

## CF constants summary

A number of constants are defined for convenience. You use them in API calls and for comparison with current `CF` variables values:

* [CF.GlobalTokensJoin](gui.md#cF.GlobalTokensJoin) join name to use to access global tokens
* [CF.PortraitOrientation](display.md#cF.PortraitOrientation) device Portrait orientation
* [CF.LandscapeOrientation](display.md#cF.LandscapeOrientation) device Landscape orientation
* [CF.AnimationCurveLinear](gui.md#animation_curve_constants): Animation constant for [CF.setProperties()](gui.md#cF.setProperties). Changes are interpolated linearly over the requested duration
* [CF.AnimationCurveEaseIn](gui.md#animation_curve_constants): Animation constant for [CF.setProperties()](gui.md#cF.setProperties). Changes begin slowly then accelerate towards the end of the requested duration
* [CF.AnimationCurveEaseOut](gui.md#animation_curve_constants): Animation constant for [CF.setProperties()](gui.md#cF.setProperties). Changes begin then slow down towards the end of the requested duration
* [CF.AnimationCurveEaseInOut](gui.md#animation_curve_constants): Animation constant for [CF.setProperties()](gui.md#cF.setProperties). Changes begin slowly, accelerate in the middle then slow down again towards the end of the requested duration
* [CF.UTF8](net.md#Constants) UTF-8 conversion type for [CF.send()](net.md#cF.send)
* [CF.BINARY](net.md#Constants) Binary conversion type for [CF.send()](net.md#cF.send)

Hash type constants for [CF.hash()](util.md#cF.hash):

* [CF.Hash_MD5](util.md#hash_type_constants)
* [CF.Hash_SHA1](util.md#hash_type_constants)
* [CF.Hash_SHA256](util.md#hash_type_constants)
* [CF.Hash_SHA384](util.md#hash_type_constants)
* [CF.Hash_SHA512](util.md#hash_type_constants)

CRC type constants for [CF.crc()](util.md#cF.crc):

* [CF.CRC_8](util.md#cRC_type_constants)
* [CF.CRC_16](util.md#cRC_type_constants)
* [CF.CRC_16_CCITT](util.md#cRC_type_constants)
* [CF.CRC_16_MODBUS](util.md#cRC_type_constants)
* [CF.CRC_32](util.md#cRC_type_constants)
* [CF.CRC_32C](util.md#cRC_type_constants)

Output format constants for [CF.crc()](util.md#cF.crc):

* [CF.OUTPUT_NUMBER](util.md#output_format_constants)
* [CF.OUTPUT_STRING](util.md#output_format_constants)
* [CF.OUTPUT_BINARY](util.md#output_format_constants)
* [CF.OUTPUT_BINARY_LE](util.md#output_format_constants)

Special item index values for [CF.listUpdate()](lists.md#cF.listUpdate)

* [CF.LastItem](lists.md#item_index_constants)
* [CF.AllItems](lists.md#item_index_constants)

List scroll position constants for [CF.listScroll()](lists.md#cF.listScroll)

* [CF.TopPosition](lists.md#list_scroll_position_constants)
* [CF.MiddlePosition](lists.md#list_scroll_position_constants)
* [CF.BottomPosition](lists.md#list_scroll_position_constants)
* [CF.VisiblePosition](lists.md#list_scroll_position_constants)
* [CF.RelativePosition](lists.md#list_scroll_position_constants)
* [CF.PixelPosition](lists.md#list_scroll_position_constants)

Sensor constants - see [Sensors](sensors.md) for more info

* [CF.Accelerometer](sensors.md#constants)
* [CF.Gyroscope](sensors.md#constants)
