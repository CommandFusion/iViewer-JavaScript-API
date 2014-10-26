# Utilities API

## Events

#### CF.DevicePropertyChangedEvent

This event is fired when one of the supported device properties changes. You can be notified when the screen brightness, sound output volume, battery level and battery charge status changes. Use `CF.watch` to start watching the changes. Your callback function is called with two parameters:

* the name of the property that is changing (allowing you to use a single callback function to watch multiple properties)
* the new value of the property.

Example:

    CF.userMain = function() {
        // Start watching all properties
        CF.watch(CF.DevicePropertyChangeEvent, CF.ScreenBrightnessProperty, onPropertyChange);
        CF.watch(CF.DevicePropertyChangeEvent, CF.SoundOutputVolumeProperty, onPropertyChange);
        CF.watch(CF.DevicePropertyChangeEvent, CF.BatteryLevelProperty, onBatteryChange);
        CF.watch(CF.DevicePropertyChangeEvent, CF.BatteryChargeStatusProperty, onBatteryChange);
    };
    
    function onPropertyChange(property, value) {
        if (property == CF.ScreenBrightnessProperty) {
            CF.log("New screen brightness: " + value);
        } else if (property == CF.SoundOutputVolumeProperty) {
            CF.log("New sound volume: " + value);
        }
    }
    
    function onBatteryChange(property, value) {
        if (property == CF.BatteryLevelProperty) {
            var percent = Math.floor(value * 100.0);
            CF.log("Battery level: " + percent + "%");
        } else {
            if (value == CF.CHARGE_UNPLUGGED) {
                CF.log("Device is now unplugged");
            } else if (value == CF.CHARGE_CHARGING || value == CF.CHARGE_FULL) {
                CF.log("Device is now plugged");
            }
        }
    }


See also [CF.setDeviceProperty()](#cF.setDeviceProperty).


## Constants

#### Device property constants

These constants are to be used to watch `CF.DevicePropertyChangeEvent` for specific properties. See [CF.setDeviceProperty()](#cF.setDeviceProperty) for an example of use.

* `CF.ScreenBrightnessProperty` refers to the screen brightness that can by changed through [CF.setDeviceProperty()](#cF.setDeviceProperty) and can be read from [CF.device.screenBrightness](globals.md#cF.device). When monitoring screen brightness changes, this event will fire if the user changes the device brightness from the Settings panel, or if the device auto-adjusts brightness according to current lighting conditions. On iOS devices, the event is fired only on iOS 5 and later.
* `CF.SoundOutputVolumeProperty` refers to the current sound output volume that can be read from [CF.device.soundOutputVolume](globals.md#cF.device)
* `CF.BatteryLevelProperty` refers to the device's battery charge level that can be read from [CF.device.batteryLevel](globals.md#cF.device).
* `CF.BatteryChargeStatusProperty` refers to the device's current charging status (unplugged, charging, full) that can be read from [CF.device.batteryChargeStatus](globals.md#cF.device)

#### Hash type constants

* `CF.Hash_MD5` is the `hashType` parameter for computing MD5 hashes with `CF.hash()`
* `CF.Hash_SHA1` is the `hashType` parameter for computing SHA-1 hashes with `CF.hash()`
* `CF.Hash_SHA256` is the `hashType` parameter for computing SHA-256 hashes with `CF.hash()`
* `CF.Hash_SHA384` is the `hashType` parameter for computing SHA-384 hashes with `CF.hash()`
* `CF.Hash_SHA512` is the `hashType` parameter for computing SHA-512 hashes with `CF.hash()`


#### CRC type constants

The CRC constants should be used for the `crcType` parameter to `CF.crc()`. Select the right one according with the type of CRC you want to generate:

* `CF.CRC_8`: 8-bit CRC
* `CF.CRC_16`: 16-bit CRC
* `CF.CRC_16_CCITT`: 16-bit CRC (CCITT polynom parameters)
* `CF.CRC_16_MODBUS`: 16-bit CRC (for MODBUS)
* `CF.CRC_32`: 32-bit CRC
* `CF.CRC_32C`: 32-bit CRC variant


#### Output format constants

The following output format constants determine the generated format for a CRC:

* `CF.OUTPUT_NUMBER`: your callback receives a number
* `CF.OUTPUT_STRING`: your callback receives a string of ASCII representation of hex CRC
* `CF.OUTPUT_BINARY`: your callback receives a string of binary bytes in network (big endian) format (for > 8-bit CRCs)
* `CF.OUTPUT_BINARY_LE`: your callback receives a string of bytes in little-endian format (for > 8-bit CRCs)

#### Battery charge constants

These constants refer to both the [CF.device.batteryChargeStatus](globals.md#cF.device) variable (updated by iViewer) and to the value you receive in your callback when monitoring [CF.BatteryChargeStatusProperty](#device_property_constants):

* `CF.CHARGE_UNKNOWN`: the charging status of the device could not be determined.
* `CF.CHARGE_UNPLUGGED`: the device is currently unplugged.
* `CF.CHARGE_CHARGING`: the device is plugged and charging.
* `CF.CHARGE_FULL`: the device is plugged and its battery is fully charged.


## Functions

#### CF.log(msg)

General logging function that logs strings to either the Remote Debugging Monitor  console (if connected), or to the webview on the current page that is marked as _Use for script debugging output_.

#### CF.logObject(object)

Dumps the details contents of an object. In JavaScript, the generic object.toString() method returns "[Object object]" for objects, which is not what you want to see the contents (all properties) of an object.

#### CF.crc(crcType, string, outputFormat, callback)

Compute a CRC for the given string. The `crcType` constants can be one of the `CF.CRC_*` constants listed in the Constants section. You pass a string to compute the CRC of, and an `outputFormat` that is one of the `CF.OUTPUT_*` contents listed in the Constants section.

Examples:

    var s = "Hello, world!";

    // Compute a 16-bit CRC, set a join with the string result
    CF.crc(CF.CRC_16, s, CF.OUTPUT_STRING, function(crc) { CF.setJoin("s1", "The CRC is: 0x" + crc) });
	
	// Compute a 32-bit CRC and assemble a binary packet sent to an external system
	// the packet header and footer string generation is not shown here
	CF.crc(CF.CRC_32, s, CF.OUTPUT_BINARY, function(crc) {
	    var cmd = somePacketHeader + crc + s + somePacketFooter;
	    CF.send("TEST-SYSTEM", cmd);
	});



#### CF.hash(hashType, string, callback)

Compute a hash an call your callback function with the hash string. The `hashType` format should be one of the `CF.Hash_*` constants listed in the Constants section.

Example:

    // Get the string from s701, compute several hashes then put the result in other joins
    CF.getJoin("s701", function(join, value) {
    	CF.hash(CF.Hash_MD5, value, function(hash) { CF.setJoin("s710", hash) });
    	CF.hash(CF.Hash_SHA1, value, function(hash) { CF.setJoin("s711", hash) });
    	CF.hash(CF.Hash_SHA256, value, function(hash) { CF.setJoin("s712", hash) });
    	CF.hash(CF.Hash_SHA384, value, function(hash) { CF.setJoin("s713", hash) });
    	CF.hash(CF.Hash_SHA512, value, function(hash) { CF.setJoin("s714", hash) });
    });


#### CF.openURL(url)

Opens a URL, the same way iViewer does it when a button with an associated URL is being pressed. The URL can be a remote URL (http://, etc) which will trigger an application switch from iViewer to the device web browser. It can also be a URL with a specific scheme recognized by the OS, so as to open another application that supports URL schemes.

Examples:

    // Open the apple.com web page directly in the web browser. If multitasking is turned off, this will quit iViewer
    CF.openURL("http://www.apple.com");
    
    // Call a phone number (iPhone) using the Phone application's URL scheme
    // (note that phone numbers must have extra chars like spaces and parenthesis removed)
    CF.openURL("tel:6501489654");
    
    // Open the Mail application (iPhone, iPad) with pre-filled fields
    var subject = "iViewer Rocks";
    var body = "Dear CommandFusion team,\n\n We love iViewer JavaScript!";
    var address = "support@commandfusion.com";
    CF.openURL("mailto:" + address + "?subject=" + encodeURIComponent(subject) + "&body=" + encodeURIComponent(body));

#### CF.loadGUI(url, settings)

Loads another GUI (or reload the current GUI). This functionality was previously available using [CF.openURL("cf://<url>")](#cF.openURL). This specialized API lets you override the current settings to customize them. In particular, you can use settings customization to force a reload of the GUI and / or assets even if the current settings don't mandate it.

The given `url` can be a real URL, an old-style string with the `cf://` scheme (same as using a straight URL, it was kept for compatibility reasons) or an iViewer 5 profile name using the `profile:` scheme. See examples below.

If you pass `null` or an empty string for the `url` parameter, the current GUI will be reloaded.

The `settings` parameter is an object which contains a number of properties that override the current settings. If a property is missing, the current setting will be used instead. Supported properties are:

* `reloadGUI` (_boolean_): force the "Reload GUI" setting to either `true` or `false`
* `reloadAssets` (_boolean_): force the "Reload Assets" setting
* `preloadAllAssets` (_boolean_): force the "Preload Images" setting
* `rememberLastGUI` (_boolean_): force the "Remember Last GUI File" setting
* `buttonPressSound` (_boolean_): force the "Button Press Sound" setting
* `enableMultitasking` (_boolean_): force the "Enable Multitasking" setting
* `enableProximitySensor` (_boolean_): force the "Proximity Sensor" (iPhone only) setting
* `disconnectOnSuspend` (_boolean_): force the "Disconnect on Exit/Sleep" setting
* `showPreloadStatus` (_boolean_): force the "Show Preload" setting (display pre-caching progress)
* `autoLockDelay` (_number_): force the "Auto Lock Delay" setting (number is expressed as a number of minutes)
* `password` (_string_): password to use for connection to the Main Control System
* `startPage` (_string_): name of the page to display after loading the GUI. If page doesn't exist, the default start page will show instead (**added in iViewer 5**)

Examples:

	// Restart the current GUI, using all the current settings unmodified
	CF.loadGUI("");

	// Reload the current GUI and all assets from server then restart it
	CF.loadGUI("", { reloadGUI: true, reloadAssets: true });

	// Load another GUI, ensure all assets are preloaded
	CF.loadGUI("http://192.168.0.100/my.gui", { preloadAllAssets: true });
	
	// Load a predefined iViewer 5 profile and go straight to a page named Theater
	CF.loadGUI("profile:My profile name", { startPage: "Theater"} );



#### CF.loadAsset(assetName, dataEncoding, callback, cache)

Loads an asset file from the designated assets folder of your GUI (or from the same folder as the GUI is no assets folder was defined). The data can be read either as binary (using `dataEncoding` value of `CF.BINARY`) or as a UTF-8 string (using `dataEncoding` value of `CF.UTF8`). Once the load is complete, your callback function is called with the asset file content. The callback function is optional: if you are using CF.loadASset() only to pre-cache asset, you may omit it.

This API can be used to:

* Load files that come along with your GUI and you want to use from JavaScript (for example, JSON data files, etc)
* Load files from external sources, quickly getting a local copy from the cache (unlike [CF.request](net.md#cF.request), you can control caching)
* Ensure that images are present in the cache for future use, for example for use in lists
* Clear images from the cache and reload them.

If the GUI was originally loaded from a server and the asset is not found in the cache, iViewer will request the asset from the server, relative to the GUI URL and optionally to the asset folder at this location that you defned in guiDesigner, just like it does for other assets. If you provide a full URL, iViewer can still look in the cache first then fallback to downloading the asset (see cache options below).

Note that if your GUI was loaded in the form of a zip file, iViewer won't try to load an asset from the server if it was not found in the zip file. To load additional assets, you will have to provide a full URL.

The `cache` mode controls cache management for this asset:

* omitted or `CF.CACHE_READONLY`: iViewer will try to load the asset from the cache, and if missing will proceed to download it but will **not** store the downloaded file back in the cache.
* `CF.CACHE`: iViewer will try to load the asset from the cache. If missing, will download it from the specified URL, and store the result in the cache.
* `CF.NO_CACHE`: iViewer will never look into the cache and always try downloading the asset. Resulting data will *not* be stored back into the cache. This can be used to completely bypass the cache and ensure a fress copy
* `CF.RECACHE`: iViewer will clear any existing local cache copy, load from the requested URL and store the resulting data back in the cache.
* `CF.DECACHE`: this is a special mode that will only wipe an asset from the local cache, but not reload it. Therefore, even if you provide a callback function it will not be called.

Note that since extra assets are not themselves defined in the GUI, they are not subject to preloading like other assets (images, sounds, videos). If you need extra assets for your Javascript code, you may want to consider providing your GUI as a *zip* file.

Here is an example flow of what happens when you use CF.loadAsset:

![CF.loadAsset flow](/assets/CF.loadAsset.png "CF.loadAsset() flow")


Examples:

    // Load an IR codes definition file from our assets. The file is defined as JSON, so we
    // can parse it with JavaScript JSON parser. The file must available on the server that provided the GUI, at the
    // same location as other assets, as defined in the guiDesigner project
    CF.loadAsset("ircodes.json", CF.UTF8, function(assetFileAsString) {
        // We receive the whole file as a string. Since we specified a UTF8 format, the
        // string can be immediately used
        var ircodesObject = JSON.parse(assetFileAsString);
        // ... do something with the ircodesObject ...
    });

    // Ensure an image is in the cache. We don't care about getting the result, just need to make sure
    // that this asset (not directly referenced by an image in a page) is in cache
    CF.loadAsset("someImage.png", CF.BINARY, null, CF.CACHE);

    // Load an external asset into the cache for future use (for example in a list)
    CF.loadAsset("http://mysite.com/myImage.png", CF.BINARY, null, CF.CACHE);

    // Reload a cached copy of some asset. If it was locally cached, the local copy is wiped then
    // a new one is downloaded
	CF.loadAsset("someImage.png", CF.BINARY, null, CF.RECACHE);

	// Pre-cache multiple assets
	CF.loadAsset(["image1.png","image2.png","http://mysite.com/someImage.png"], CF.BINARY, null, CF.CACHE);


Compatibility notes:

A previous version of this API did not include the `cache` mode. It is still supported, and by default will operate with the `CF.CACHE_READONLY` mode.


#### CF.setDeviceProperty(property, value)

Change a device property to a new value. The only property currently supported by this call is `CF.ScreenBrightnessProperty` (on iOS 5 and later, and on Android), allowing you to tune the brightness of the device display. You can't turn it completely off: even if you set the brightness to 0.0 the backlight will stay somewhat visible.

The `CF.ScreenBrightnessProperty` value is a number between 0.0 and 1.0 (inclusive). The current screen brightness is being reflected by the [CF.device.screenBrightness](globals.md#cF.device) property. Changes can be observed by watching the `CF.DevicePropertyChangedEvent` with the `CF.ScreenBrightnessProperty`.

Note that on iOS, changes in display brightness triggered by the application do not generate a `CF.DevicePropertyChangedEvent`.

Example:

    // Set the screen brightness to half
    CF.setDeviceProperty(CF.ScreenBrightnessProperty, 0.5);
