# Application and GUI API

## Events

#### CF.PreloadingCompleteEvent

This event is sent once by iViewer after your GUI loads, your CF.userMain() function has completed, external systems are started and, if you turned on preloading, caching images and sounds is done. You can use this event if you want to launch network-intensive operations but need to wait that iViewer is done preloading your images, so to provide a responsive UI with all the images already loaded.

Example:

    CF.userMain = function() {
        CF.watch(CF.PreloadingCompleteEvent, onPreloadingComplete);
    };
    
    function onPreloadingComplete() {
        CF.unwatch(CF.PreloadingCompleteEvent);
        // here you can start your network-intensive operations,
        // the GUI finished all its preloading.
    }


#### CF.JoinChangeEvent

This event is sent when a join value changes. With a single call to `CF.watch()`, you can start watching multiple joins at once. When the even is fired for a join change, your callback receive three parameters:

* `join` (_string_): the join that changed
* `value` (_string_): the new join value
* `tokens` (_object_): the tokens associated with the join
* `tags` (_array_): an array of strings (can be empty) that lists the tags associated with this object

The event-specific parameters to `CF.watch()` can either be a join string, or an array of join strings. As with other events, you can use `CF.unwatch()` to stop watching this event.

Example:

    // Common function that receives the join changes we watch
    function onJoinChanged(join, value, tokens, tags) {
        CF.log("Join " + join + " changed to: " + value);
    }

    CF.userMain = function() {
        // Start watching changes to a1
        CF.watch(CF.JoinChangeEvent, "a1", onJoinChanged);

        // Start watching changes to several joins
        CF.watch(CF.JoinChangeEvent, ["s1", "d4", "d5", "a12"], onJoinChanged);

        // Stop watching the changes. Note that we must use the exact same event-specific parameters
        // to properly unwatch
        CF.unwatch(CF.JoinChangeEvent, "a1");
        CF.unwatch(CF.JoinChangeEvent, ["s1", "d4", "d5", "a12"]);
    };


#### CF.InputFieldEditedEvent

This event is sent during input fields edition, whenever an edit action changes the value of the field (text entry, text deletion, cut, paste). By observing this event, your JavaScript can achieve advanced user interfaces like type-ahead selection in lists (scroll a list to the entry closest to the name that's currently in the field), etc.

The callback you provide receives the same parameters as the callback for `CF.JoinChangeEvent`:

* `join` (_string_): the serial join of the input field that was edited
* `value` (_string_): the new input field contents
* `tokens` (_object_): the tokens associated with the input field join
* `tags` (_array_): an array of strings (can be empty) that lists the tags associated with this object

As with other events, you can use `CF.unwatch()` to stop watching this event.


Example:

    CF.userMain = function() {
        CF.watch(CF.InputFieldEditedEvent, "s1", function(join, value, tokens, tags) {
            CF.log("Input field " + join + " just changed to value " + value);
        });
    };
    
    function stopWatchingInputField() {
        // At any time, your code can stop observing the CF.InputFieldEditedEvent
        CF.unwatch(CF.InputFieldEditedEvent, "s1");
    }


#### CF.KeyboardUpEvent

This event is sent whenever the soft keyboard pops up on the device. You can take action, like hiding subpages or objects, sending information to a remote system, etc. The callback receives the join of the field being edited as a parameter.

See [CF.KeyboardDownEvent](#cF.KeyboardDownEvent) for a usage example.

#### CF.KeyboardDownEvent

This event is sent whenever the soft keyboard slides down on the device. The callback receives the join of the field that was being edited as a parameter, as well as the field contents after edit.

Example:

    CF.userMain = function() {
        CF.watch(CF.KeyboardUpEvent, onKeyboardUpEvent);
        CF.watch(CF.KeyboardDownEvent, onKeyboardDownEvent);
    };
    
    function onKeyboardUpEvent(editedField) {
        CF.log("Keyboard is up, editing field of join " + editedField);
    }
    
    function onKeyboardDownEvent(editedField, fieldText) {
        CF.log("Keyboard is down, done editing field of join " + editedField + " with text: '" + fieldText + "'");
    }


#### CF.ObjectPressedEvent

This event is sent when a button of slider is pressed. Your callback function receives the join string of the pressed item, as well as its current value and the object's tokens. The value may not be up to date if the object is not in Simulation mode, and the join update has not yet been received from the Control System. In all cases, the fact that you receive `CF.ObjectPressedEvent` indicates that the button or slider has just been tapped.

You can pass one join string to watch, or an array with multiple joins. Note that sliders send events only on their analog join.

This event can be useful in situations where you want to be notified immediately of a button press, regardless of the roundtrip time it takes for the control system to send the join update back to iViewer.

Example:

    function onItemPressed(join, value, tokens) {
        CF.log("GUI item " + join + " pressed");
    }

    CF.userMain = function() {
        // Watch a button
        CF.watch(CF.ObjectPressedEvent, "d5", onItemPressed);
        
        // Watch two sliders and a button
        CF.watch(CF.ObjectPressedEvent, ["d5", "a8", "a9"], onItemPressed);
    };


#### CF.ObjectDraggedEvent

This event is sent by sliders when the slider is being dragged by the user. The event is never sent for buttons. The value you receive is the current value of the slider, regardless of join updates that may come later from the Control System (the value is computed during the drag, then regularly transmitted to the Control System, according to the transmit interval you have defined in your GUI). This event allows watching and getting immediate, live updates during a slider drag with no delay.

Note that sliders send events only on their analog join.

Example:

    function onSliderPressed(join, value, tokens, tags) {
        CF.log("Begin dragging slider " + join + ", current value=" + value);
    }
    
    function onSliderDragged(join, value, tokens, tags) {
        CF.log("Moving slider " + join + ", new value=" + value);
    }
    
    function onSliderReleased(join, value, tokens, tags) {
        CF.log("End dragging slider " + join + ", final value=" + value);
    }
    
    CF.userMain = function() {
        // Watch slider a1
        CF.watch(CF.ObjectPressedEvent, "a1", onSliderPressed);
        CF.watch(CF.ObjectDraggedEvent, "a1", onSliderDragged);
        CF.watch(CF.ObjectReleasedEvent, "a1", onSliderReleased);
    };


#### CF.ObjectReleasedEvent

This event is sent when user lifts the finger from a button or a slider. For buttons, your callback receives the current join value of the button (which may not be up to date, depending on whether the button is set to Simulation mode, or whether the Control System has already sent the join update to iViewer). For sliders, you receive the current value of the slider, regardless of whether the value has already been transmitted and further received from the Control System.

See `CF.ObjectDraggedEvent` above for an example of use with sliders.

#### CF.GUISuspendedEvent

This event is sent to your script in the following occasions:

* Under some circumstances when the OS temporarily takes over the user interface (for example, when receiving a SMS)
* When iViewer is put in background on the device (user switches applications) if multitasking is enabled
* When iViewer quits (switching applications with multitasking disabled)

Beware that you are allowed **very little time** to do any processing here. You should refrain from performing any lengthy processing in this situation, but can use the occasion to set some variables (usually, remember the time at which application goes into background, so as so take appropriate action depending on the time spent outside of iViewer when coming back to it).

Additionally, any call you make into iViewer at this time (all `CF.*` calls) will not be performed immediately. Rather, it will be performed when the application resumes. Because of this, you should not try to make any call into CF when processing this event.

In practice, you will only want to perform small Javascripts tasks at this stage, and do real processing later when `CF.GUIResumedEvent` is sent.

Note that contrary to most other events, asking an initial fire of `CF.GUISuspendedEvent` by adding a last `true` parameter to the `CF.watch` call is not supported.

Example:

    function onGUISuspended() {
        // Even though the call is not executed immediately, it is enqueued for later processing:
        // the displayed date will be the one generated by the time the app was suspended
        CF.log("GUI suspended at " + (new Date()));
    }
    
    function onGUIResumed() {
        // Show the time at which the GUI was put back to front
        CF.log("GUI resumed at " + (new Date()));
    }

    CF.userMain = function() {
        CF.watch(CF.GUISuspendedEvent, onGUISuspended);
        CF.watch(CF.GUIResumedEvent, onGUIResumed);
    };


#### CF.GUIResumedEvent

This event is sent to your script when the application resumes (and always after `CF.GUISuspendedEvent`) when:

* iViewer becomes active after the OS temporarily put it in the background (for example, an SMS received alert was displayed, the multitasking bar was shown, etc)
* the user switches back to iViewer (multitasking enabled)

At this time, the application is returning the active status. Any call into iViewer that you may have made when receiving a `CF.GUISuspendedEvent` will be executed.



## Constants

#### CF.guiURL

The GUI URL from which we downloaded the GUI (if the GUI was read from local cache, the URL still reflects the original location of the GUI).

#### CF.GlobalTokensJoin

Global tokens are being stored in a special join named `e0`. When you want to access global tokens, you can do so using `CF.getJoin()`, `CF.getJoins()`, `CF.setJoins()` and `CF.setTokens()`. Instead of using a regular join number, use the `CF.GlobalTokensJoin` constant. Do not use `"e0"`, as the way global tokens are accessed may change in the future. Using `CF.GlobalTokensJoin` guarantees compatibility of your code with future versions of iViewer.

#### Animation curve constants

The [CF.setProperties()](#cF.setProperties) function let you change visual properties of join elements in your GUI, with optional animation. When you specify an animation duration, you can also specify an animation curve:

* `CF.AnimationCurveLinear`: changes are interpolated linearly over the requested duration
* `CF.AnimationCurveEaseIn`: changes begin slowly then accelerate towards the end of the requested duration
* `CF.AnimationCurveEaseOut`: changes begin then slow down towards the end of the requested duration
* `CF.AnimationCurveEaseInOut`: changes begin slowly, accelerate in the middle then slow down again towards the end of the requested duration

## Functions


#### CF.getJoin(join, callback)

Get the current value and tokens of a join. Due to the asynchronous nature of the bridge between iViewer and your script, you can't read the join value directly. Rather, you have to provide a callback function that will receive the value and tokens for this join.

Joins should be in the usual form "**type**_number_", for example `"s1"`, `"a100"`, etc. You can access joins that are in list elements, using the list join access syntax: "_list join_**:**_index_**:**_join_". Remember that list indices are 0-based. To get the value of join `"s1"` in the third item of list `"l1"`, you will therefore use: `"l1:2:s1"`.

Use `CF.GlobalTokensJoin` to access the join which keeps the global tokens. In this case, the value will always be an empty string.

Regardless of the join type (serial, analog, digital or list), the returned value is always a string. For analog joins, you can may want to use Javascript's `parseInt()` and `parseFloat()` functions to process the value as an actual number.

Your callback will receive three parameters:

* `join` (_string_): the join that was requested
* `value` (_string_): the current join value
* `tokens` (_object_): an object containing all the join's tokens as properties
* `tags` (_array_): an array of strings (can be empty) that lists the tags associated with this object

If the requested join does not exist, your callback will receive a `null` value and `null` tokens object. You can use this when writing common code modules that are used in multiple situations, where you want to check at runtime if specific elements of your GUI are present.

To read multiple join values at once, it is more efficient to use the `CF.getJoins()` function.

As with all other calls, you can either use a separate function or directly write the callback function as a parameter to the `CF.getJoin()` call (this is called a _lambda_ function, or more commonly _closure_). Writing code this way improves the readability of your code, as getting a join is always followed by some action on the received value.

Examples:

    // Get the value of a100
    CF.getJoin("a100", function(join, value, tokens) {
        // regardless of the join type, the value is always a string
        // if we want a number, we need to use parseInt() or parseFloat()
        var num = parseFloat(value);
        if (num > 10.0) {
            CF.log("a100 is above 10");
        } else {
            CF.log("a100 is below 10");
        }
    });

    // Test whether the GUI is configured with join s1000. Note that since we are
    // not using the tokens parameter of the callback, we are free to
    // omit it.
    var hasS1000 = false;
    CF.getJoin("s1000", function(join, value) {
        if (value != null)
            hasS1000 = true;
    });
    
    // Check some global tokens
    CF.getJoin(CF.GlobalTokensJoin, function(j, v, tokens, tags) {
        // read a token named [currentartist]
        var currentArtist = tokens["[currentartist]"];
        
        // read a token named SERVER_URL
        var serverURL = tokens["SERVER_URL"];
        
        // ... now do something with these variables
    });


#### CF.getJoins(joins, callback)

Get the current value and tokens for multiple joins. This variant of `CF.getJoin()` is more efficient, as it incurs only one roundtrip between your script and iViewer to get several joins at once. You pass an array of joins to get, and your callback receives an object for which:

* each requested join is a property with the same name
* the information about the join is itself an object, with properties:
    * `value` (_string_): the value of the requested join, as a string
    * `tokens` (_object_): an object which carries the token names as properties, and string value as the value for each property.
	* `tags` (_array_): an array of strings (can be empty) that lists the tags associated with this join

The same rules as for `CF.getJoins()` apply:

* values are always returned as a string. Convert to number when needed using `parseInt()` or `parseFloat()`,
* the global tokens join can be read using the join identifier `CF.GlobalTokensJoin`,
* missing joins (joins that don't exist in the GUI) will have a `null` _value_ and `null` _tokens_ properties,
* you can request join values in list elements using the "_list join_**:**_index_**:**_join_" syntax.

Example:

    // Get three joins at once
    CF.getJoins(["s1", "s2", "a3"], function(joins) {
        // set s3 to be s1+s2 strings
        CF.setJoin("s3", joins["s1"].value + joins["s2"].value);
        
        // Check whether a3 exists in this GUI
        if (joins["a3"].value == null) {
            CF.log("Join a3 doesn't exist in this GUI");
        }
    });


##### Advanced tips and pitfalls for CF.getJoins()

Javascript being flexible, you can access the returned joins using either the array syntax (`joins["s1"]`) or the property syntax (`joins.s1`). The later form is easier to use, and works equally. In cases where your build the join string dynamically (for example, `var someJoin = "s" + joinNumber;`) you will use only the array syntax. In this case, the property syntax (`joins.someJoin`) would really try to access `joins["someJoin"]` which is not what you want. Instead, just write `joins[someJoin]` which does the expected job of taking the value of variable `someJoin` and using accessing a property carrying the name contained in `someJoin`.

    // Dynamically building and accessing joins
    var index = 18;
    var num = 100;
    var join1 = "s" + num;                  // join1 = "s100"
    var join2 = "l1:" + index + ":s" + num; // join2 = "l1:18:s100"
    
    // Get the values of "s100", "l1:18:s100" and "s1"
    CF.getJoins([join1, join2, "s1"], function(joins) {
        // CORRECT: retrieve the value of s1 using the property syntax
        CF.log("The value of s1 is: " + joins.s1.value);
        
        // CORRECT: retrieve the value of s1 using the array syntax
        CF.log("The value of s1 is: " + joins["s1"].value);
        
        // INCORRECT: trying to retrieve the value of join1 using the property syntax
        // This would really try to access joins["join1"].value, which does not exist
        CF.log("No luck with retrieving s100: " + joins.join1.value);
        
        // CORRECT: retrieve the value of join1 using the array syntax
        CF.log("The value of s100 is: " + joins[join1].value);
        
        // CORRECT: retrieve the value of join2 using the array syntax
        CF.log("The value of l1:18:s100 is: " + joins[join2].value);
    });


#### CF.setJoin(join, value [, sendJoinChangeEvent])

To set the value of a join, simply use `CF.setJoin()`. You may pass the value as a boolean, a number or a string. If you want to set multiple joins at once, you can use `CF.setJoins()` which is faster than multiple calls to `CF.setJoin()`. By default, `CF.setJoin()` fires a `CF.JoinChangeEvent`. You can change this behavior by passing an additional parameter `false`. This way, you can ensure that your watcher function is not called.

When setting digital joins, any non-zero value that you pass (after potential conversion from string to number) will set the digital join to 1.

When setting strings, the strings are evaluated by iViewer the same way they are when processing feedback. This allows using both math expressions and references to other joins. This way, you can directly place the result of small computations without having to fetch the values of other joins.

Example:

    CF.setJoin("s1", "Hello, world!");

    // equivalent forms
    CF.setJoin("a1", 226.12);
    CF.setJoin("a1", "226.12");     // converts string to 226.12

    CF.setJoin("s2", "175");
    CF.setJoin("s2", 175);          // converts number to string

    CF.setJoin("d1", 1);
    CF.setJoin("d1", 10);           // clips to 1
    CF.setJoin("d1", "1");          // string -> number [0, 1]
    CF.setJoin("d1", "test");       // string to number evaluates to 0 -> 0
    
    // computations and join references
    CF.setJoin("s1", "Living Room");
    CF.setJoin("s2", "In [@s1] zone");  // Will set s2 to "In Living Room zone"
    CF.setJoin("s3", "Current level: {{floor([@a1] / 10)}}");   // will set s3 to "Current level: 22"
    
    // Preventing fire of CF.JoinChangeEvent
    CF.setJoin("s1", "Living Room", false);     // won't fire CF.JoinChangeEvent


#### CF.setJoins(array [, sendJoinChangeEvent])

Rather than repeatedly calling `CF.setJoin()` to set several joins, it is more efficient to call `CF.setJoins()` once and provide it with an array of the joins to set. By defaults, `CF.setJoins()` will send a `CF.JoinChangeEvent` for each join change. You can pass an optional additional parameter `false` to prevent iViewer from notifying your script of the joins changes. If the `sendJoinChangeEvent` parameter is missing, it defaults to `true`.

Each element of the array must be an object which should have the following properties:

* `join` (_string_): the join to set (can also use the list element syntax)
* `value` (_string_): if present and not `null`, the value to set (numbers and booleans are also accepted, they will be converted)
* `tokens` (_object_): if present and not `null`, and object whose properties values will set tokens values named after the properties.

Regarding the expected format and conversion of values, including math expressions and join references, the same rules as in `CF.setJoin()` apply. Additionally, all joins are set in the order they appear in the array, so if a join value references the value on another join which is being also set in the same array, the new value of the join will be used if the reference is made in a join set that appears after the referenced one.

If a `join` property is `CF.GlobalTokensJoin`, and you pass a `tokens` property, the tokens will be set globally (global tokens).

Example:

    CF.setJoins([
        { join:"s1", value:"Hello, world!" },   // set string value
        { join:"a1", value:18 },                // set number on analog join
        { join:"a2", value:10, tokens: {        // set the value of a2, as well as two tokens
            "[count]": count,
            "max": 18,
        }},
        { join:"a3", value:"{{[@a1] + [@a2]}}" }, // set a3 to be the sum of a1 and a2. New values being set prior to this one, the value of a3 will be 28.
        { join:"s5", tokens: { "albumID": "43ABF998D" } },
        { join:CF.GlobalTokensJoin, tokens: { "[current-zone]": "Living Room" }}   // set a global token
    ]);

With a little care and experience, you can easily construct arrays of values to set programmatically and update several parts of your GUI at once.


#### CF.setToken(join, token, value)

To set a single token on a single join, you can use `CF.setToken()`. Pass `CF.GlobalTokensJoin` join identifier to set a global token. Tokens go through the same evaluation mechanism as values. Therefore, you can use embedded math expressions and join references in the token value.

Similarly, you can reference list elements by using the list join reference syntax: "_list join_**:**_index_**:**_join_"

Example:

    // Set token "count" on join s1. All token values are converted to strings
    CF.setToken("s1", "count", 1);
    
    // Set global token "[current-zone]"
    CF.setToken(CF.GlobalTokensJoin, "[current-zone]", "Living Room");
    
    // Set a token on join s1 in the third item of list l1
    CF.setToken("l1:2:s1", "display", "TV");

#### CF.addTag(join, tag)
#### CF.addTag(joinsArray, tag)

Associate a tag with the objects on the given join(s). All objects sharing this join will have the tag added to them. If you pass an array of join strings, all objects on joins in this array will have the tag added to them.

It is okay to add the same tag multiple times, it will be added only once. As will all other APIs using a join as parameter, it is also okay to access objects themselves by tag instead of using their join.

Example: using tags to quickly hide / show related objects

    // Add the "vol_level" tag to all objects assigned to analog join 1
    CF.addTag("a1", "vol_level");

	// Add the "group1" tag to several related objects, essentially grouping them
	CF.addTag(["d10","d11","d12","s50","s60","a10"], "group1");

	// Hide all objects assigned to the "group1" tag at once
	CF.setProperties({join: "group1", opacity: 0});


#### CF.removeTag(tag)
#### CF.removeTag(join, tag)
#### CF.removeTag(joinsArray, tag)

Remove a tag from objects with the given join. All objects sharing this join will have the tag removed. You can identify objects by any of their attached tags, you can also pass an array of strings to remove a tag from multiple objects at once (for example, you may need to remove an associated tag from only some objects, not all carrying this tag).

You can remove a tag from all the objects carrying it by simply passing the tag as sole parameter.

Examples:

	// Remove the "group1" tag from one object
	CF.removeTag("d11", "group1");

	// Remove the "group1" tag from multiple objects
	CF.removeTag(["d10","d11"], "group1");

	// Totally remove the "group1" tag from all objects carrying it
	CF.removeTag("group1");


#### CF.getProperties(join, callback)

Get the properties of a GUI element identified by its join. Your callback function will receive an object described below. If the element represented by the join lacks some of the properties that are defined below, their value will be `null`. If the element does not exist, your callback will receive `null` instead of an actual object.

Also, using `CF.getProperties` and `CF.setProperties` with GUI elements on join 0 is forbidden and will result in your callback receiving a `null` object.

Structure of the object passed to your callback by `CF.getProperties`:

* `join` (_string_): the join string
* `x` (_number_): horizontal location of the GUI element
* `y` (_number_): vertical location of the GUI element
* `w` (_number_): width of the GUI element
* `h` (_number_): height of the GUI element
* `xrotation` (_number_): the rotation angle of the GUI element on the X axis, in degrees
* `yrotation` (_number_): the rotation angle of the GUI element on the Y axis, in degrees
* `zrotation` (_number_): the rotation angle of the GUI element on the Z axis, in degrees
* `scale` (_number_): the display scale of the element, 1.0 being the default. Bigger values make the element bigger on screen, smaller values make the element smaller.
* `opacity` (_number_): opacity (alpha channel) of the GUI element, between 0.0 and 1.0. 1.0 is fully opaque, 0.0 makes the object invisible and not tapable.
* `theme` (_string_): the theme of the GUI element

You can also get and set properties on list item elements by addressing the item using the usual "l#:item#:join" format (i.e. "l1:3:s1" to get or set properties on join s1 of the fourth item in list l1).

Example:

    // Get the properties of GUI element at join d1
    CF.getProperties("d1", function(j) {
       CF.log("Join d1: x=" + j.x + " y=" + j.y + " w=" + j.w + " h=" + j.h + " opacity=" + j.opacity + " theme=" + j.theme);
    });

##### Advanced tips: getting multiple join properties at once

You can pass an array of join strings instead of a single join string. In this case, iViewer will execute your callback with an array of objects instead of a single object. Each of the objects will have the structure described above. This is much more efficient than making several successive calls to `CF.getProperties()` (and easier to manage in your script, as you have a single callback point to deal with).

Example:

    // Get the properties of 3 joins at once. We receive an array of objects
    CF.getProperties(["d1","d2","d3"], function(joins) {
       for (var i = 0; i < joins.length; i++) {
           var j = joins[i];
           CF.log("Join " + j.join + ": x=" + j.x + " y=" + j.y + " w=" + j.w + " h=" + j.h + " opacity=" + j.opacity + " theme=" + j.theme);
       } 
    });

#### CF.setProperties(changes [, delay, duration, curve, callback, ... any number of callback parameters ...])

Modify the properties of one of more GUI elements. All the properties returned by `CF.getProperties()` can be modified, and optionally animated (with the exception of theme changes, for which the change delay will be honored but not the change duration -- at the end of the delay, the theme change will trigger an immediate refresh).

To modify the properties of a single join, pass an object which has at least a `join` (_string_) property so that iViewer identifies the join to change. You are free to not pass element properties you do not wish to change. You can also directly modify the object returned by `CF.getProperties()`, change the ones you want and pass it to `CF.setProperties()`.

To modify the properties of multiple joins at once (more efficient than calling `CF.setProperties()` multiple times), pass an array of objects instead of a single object. In the array, each object's `join` (_string_) property will identify the join to modify.

With the exception of the `theme` of each join, changing all other properties can be animated. The transition will be made using an animation (if the element is on screen), starting after the specified delay (in seconds) and for a specified duration (in seconds). If you don't pass the `duration` parameter or set it to 0, the changes will occur after `delay`. If you don't pass the `delay` parameter, the changes will occur immediately. Animation is done using a linear curve.

The animation curve determines the way properties are animated. It can be one of `CF.AnimationCurveLinear`, `CF.AnimationCurveEaseIn`, `CF.AnimationCurveEaseOut`, `CF.AnimationCurveEaseInOut`.

If you specify a callback function, the callback function will be called when the animation completes, offering you the possibility to take further actions (this can be used, for example, to chain animations).

Your callback function can receive parameters, that you pass by adding any number of parameters at the end of the `CF.setProperties` call, as per in the example below. All the additional parameters to `CF.setProperties` after the callback function will be passed as arguments to the callback function itself.

Example:

    // This example takes list l1 and, assuming that each list item has an image with join s1,
    // scales up all the images with a short delay, then scales them down, providing a nice
    // "wave" effect. Note how we pass a join string to use in the callback by adding a
    // parameter to the first CF.setProperties() call.
    function doListWave(listJoin, imageJoin) {
    	CF.listInfo(listJoin, function(list, count, first, numVisible) {
    		for (var i=first, delay=0.0; i < first + numVisible; i++) {
    			var j = listJoin + ":" + i + ":" + imageJoin;
    			CF.setProperties({join: j, scale: 1.5}, delay, 0.33, CF.AnimationCurveLinear, function(joinString) {
    				CF.setProperties({join: joinString, scale: 1.0}, 0.0, 0.33, CF.AnimationCurveLinear);
    			}, j);
    			delay += 0.05;
    		}
    	});
    }
    

##### Advanced tips: Object rotations in 3D

iViewer can rotate and animate rotations of your GUI elements along the three axis (X, Y and Z), although there are some constraints you need to take into account:

* If you rotate on an axis by more than 180° at once, the rotation direction (clockwise or counter-clockwise) may be unpredictable
* To avoid visual deformation of your object, you should avoid rotating on more than two axis at once

To perform a full circle rotation on one axis, you will need to split up the rotation steps and either use completion callbacks to your `CF.setProperties()` calls, or JavaScript's `setTimeout()` function to trigger the next step of animation.

Here is an example that performs full clockwise 360° rotation on the Z axis:

    function spin360(someJoin) {
        CF.setProperties(
            {
                join: someJoin, // the object to animate
                zrotation: 179  // start with a 179° rotation to ensure it's going clockwise from 0 -> 179 (shortest path)
            },
            0.0,                // start immediately
            0.5,                // rotate for half a second
            CF.AnimationCurveLinear,
            function() {        // then call this callback
                CF.setProperties(
                    {
                        join: someJoin,
                        zrotation: 358 // 179 (current) + 179° (new, keep rotating clockwise),
                    },
                    0.0,
                    0.5,        // another half second
                    CF.AnimationCurveLinear,
                    function() {
                        // from 358° to 0°, we don't need to animate
                        CF.setProperties({join: someJoin, zrotation: 0}, 0, 0);
                    }
                );
            }
        );
    }

    
#### CF.getGuiDescription(callback)

In some cases, generic scripts need to get information about the GUI, including which joins are defined, which pages and subpages exist, etc. The `CF.getGuiDescription` call returns such basic information about the current GUI. In the v4.0 release of iViewer, only a subset of the full GUI description is included: more information will be made available in future versions as needs arise.

Only the most basic information about each object is being returned. You can get access to and modify visual properties of objects with the `CF.getProperties()` and `CF.setProperties()` calls.

Your callback receives only one parameter, an object containing properties describing the GUI. This object contains the following properties:

* `name` (_string_): the GUI name
* `url` (_string_): the GUI path or URL, as defined in the device Settings
* `portraitSize` (_object_): the size of portrait pages (an object with `w` and `h` properties for width and height)
* `landscapeSize` (_object_): the size of landscape pages (an object with `w` and `h` properties for width and height)
* `allJoins` (_array_): an array of strings with the list of all joins in the GUI
* `pages` (_array_): an array of objects for GUI pages, described below
* `subpages` (_array_): an array of objects for GUI subpages, described below

Each page in the GUI is described by an object containing the following properties:

* `name` (_string_): the page name
* `join` (_string_): the join for this page
* `type` (_string_): the string "Page"
* `portraitObjects` (_array_): an array of objects, each describing one of the GUI objects in the Portrait orientation of the page (see below)
* `landscapeObjects` (_array_): an array of objects, each describing one of the GUI objects in the Landscape orientation of the page (see below)

Each subpage in the GUI is described by an object containing the following properties:

* `name` (_string_): the subpage name
* `type` (_string_): the string "Subpage"
* `objects` (_array_): an array of objects, each describing one of the GUI objects in the subpage

Finally, each object in the GUI is described in a JavaScript object containing a subset of the actual properties:

* `join` (_string_): the join for this object
* `type` (_string_): a string with the type of the object. Can be one of `"Button"`, `"Gauge"`, `"Input"`, `"Image"`, `"Label"`, `"List"`, `"Slider"`, `"SubpageRef"`, `"Timer"`, `"Video"`, `"Webview"`
* `digitalJoin` (_string_): some object have an associated digital join, for example "Input" objects. If this is the case, this string is the join ID (otherwise an empty string).

Buttons are a special case as they have joins for their active an inactive states that can be modified to change their text. Therefore, JavaScript objects describing buttons contain additional properties:

* `activeTextJoin` (_string_): the serial join ID for active (pressed) text, or an empty string (if join was 0 in GUI Designer)
* `inactiveTextJoin` (_string_): the serial join ID for inactive (released) text, or an empty string (if join was 0 in GUI Designer)

Slider objects carry the following additional property:

* `pressedJoin` (_string_): the digital join ID that is triggered high when the slider is being pressed, or an empty string (if join was set to 0 in GUI Designer)

Video objects carry the following additional properties:

* `playTriggerJoin` (_string_): the digital join ID for the Play function of the video object, or an empty string
* `stopTriggerJoin` (_string_): the digital join ID for the Stop function of the video object, or an empty string

Webview objects carry the following additional properties:

* `backTriggerJoin` (_string_): the digital join ID for the Back trigger of the webview, or an empty string
* `forwardTriggerJoin` (_string_): the digital join ID for the Forward trigger of the webview, or an empty string
* `refreshTriggerJoin` (_string_): the digital join ID for the Refresh trigger of the webview, or an empty string
* `stopTriggerJoin` (_string_): the digital join ID for the Stop trigger of the webview, or an empty string

SubpageRef objects carry the following additional properties:

* `subpage` (_string_): then name of the subpage this object references
